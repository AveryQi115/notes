# VMagent更改思路

## 环境设置

- alg,env配置文件

  - mac
  - learner
  - agent

- 原：SchedEnv

  - 继承自gym.Env

  - env基本流程

    ```python
    initial_observation = env.reset()
    for i in range(100):
      	env.render()
      	observation, reward, done, info = env.step(action)
        if done:
          	break
    env.close()
    ```

    - step函数返回4个值
      - `observation` (**object**): 环境的观测值
      -  `reward` (**float**): 前一步动作的奖励
      - `done` (**boolean**): 当前episode是否已经结束
      - `info` (**dict**): 用于debug的信息

  - env含有两个属性，`action_space`和`observation_space`

- DeployEnv

  - 设计思路：和数据清洗模块进行对接，数据清洗模块把五分钟内的请求汇总并通过round-robin调度到pod上面更改cluster的状态，DeployEnv直接获得cluster的状态

  - $n_{node}$(**int**)

  - t(**int**)时刻序号

  - nodes(**dict**)

    - 在node层面记录node的总可分配cpu，mem，剩余可分配cpu，mem
    - 在pod层面记录pod的请求cpu，mem，实际分配到的cpu，mem
    - 设计理念按照kubernetes的Best Effort类pod分配资源，同个node上的pod按request比例分配cpu，mem按照实际请求分配

    ```json
    {
      "node1":{
        "total_cpu": 100,
        "total_mem": 100,
        "remain_cpu": 0,
        "remain_mem": 40,
    		"pods":[{
          "pod_name": "pod1",
      		"request_cpu": 50,
      		"request_mem": 60,
          "assigned_cpu": 100,
          "assigned_mem": 60,
      	},
        ...
       ]
    	},
      "node2":{
        ...
      },
      ...
    }
    ```

## 决策设置

- 原：VectorMAC
- 初始化的时候需要构造agent，提供action_space, obs_space和后面的request，observation shape相符
- TODO：更改QmixAgent类

```python
class VectorMAC:
    def __init__(self, args):
        self.args = args
        server_num, cpu_num, mem_num = args.N, args.cpu, args.mem
        action_space = server_num*2
        obs_space = [(server_num, 2, 2), (1, 2, 2)]
        self._build_agents(obs_space, action_space, args)

        self.action_selector = action_REGISTRY['epsilon_greedy'](args)
```



## 训练过程

1. 求eps(0.9,0.5,0.2,0.2)几段折线上取x对应的值，x为epoch序号

```python
for x in range(MaxEpoch):
  eps = linear_decay(x, [0, int(
            MAX_EPOCH * 0.25),  int(MAX_EPOCH * 0.9), MAX_EPOCH], [0.9, 0.5, 0.2, 0.2])
```

2. envs.reset(my_steps)

- 更改schedEnv.t，schedEnv.start
- t更新到当前或其后第一个creation command；忽略前置deletion

```python
schedEnv.t = my_steps[i]
schedEnv.start = my_steps[i]
```

3. 对于每一个epoch每一个step：

```python
# 首先获取相关state信息
avail = envs.get_attr('avail') 
# clusters' usable state, vector shape is [servers_num,2](2 for 2 numas),1 is available, 0 is not

feat = envs.get_attr('req')
# request info

obs = envs.get_attr('obs')
# clusters' current info, vector shape is [servers_num,2,2](2 for 2 numas, 2 for remain_cpu and remain_mem)

state = {'obs': obs, 'feat': feat, 'avail': avail}

# 根据相关state信息和eps选取action
action = mac.select_actions(state, eps)
```

- 选择动作

```python
# ep_batch is the got state, eps is the decay argument
def select_actions(self, ep_batch, eps):
  start = time.time()
  # compute the outputs and avail_actions by model
  agent_outputs, avail_actions = self.forward(ep_batch)
  # choose action by action_selector, such as episelon_greedy
  chosen_actions = self.action_selector.select_action(agent_outputs, eps, avail_actions, 0)
  try:
    chosen_actions.cpu().numpy()
    except:
      import pdb; pdb.set_trace()
      return  chosen_actions.cpu().numpy()
    
def forward(self, ep_batch, isDelta=False):
  # agent_inputs是tensor化的[observation,request], avail_actions are tensored available_servers
  agent_inputs, avail_actions = self._build_inputs(ep_batch)
  # QMixAgent output action
  agent_outs = self.agent(agent_inputs)
  # mask_invalid去掉结果中不符合的action
  agent_outs, avail_actions = self.mask_invalid(agent_outs, avail_actions)
  return agent_outs, avail_actions
  
def _build_inputs(self, states, is_z=False):
  obs_inputs = []
  feat_inputs = []
  if type(states) is dict:
    avail_actions = torch.Tensor(states['avail']).cuda()
    obs = torch.Tensor(states['obs']).cuda()
    feat = torch.Tensor(states['feat']).cuda()

    return [obs, feat], avail_actions
```

- 执行动作

```python
def step(self, action):
  '''
   env take action ,
   return state, reward and done
  '''
  action = self._step(action)
  if self.isrender:
    record_dict = {'server': self.cluster.describe(), 'request':self.requests[self.t], 'action': action}
    self.render_list.append(record_dict)
    self.t += 1
    self.dur += 1

    request = self.requests[self.t]
    if request['is_double'] == 1:
      request_info = [[[request['cpu']/2, request['mem']/2] for i in range(2)] for j in range(2)]
      else:
        request_info =[[[request['cpu'], request['mem']], [0,0]], [[0,0], [request['cpu'], request['mem']]]]
        self.handle_delete()
        state =  self.cluster.describe()
        reward = self.reward()
        done  = self.termination()
        return action, state, reward, done
```

- 收集total_reward

```python
mem.push(buf)
return tot_reward.mean(),tot_length.mean()
```

4. q_learner更新模型

```python
# start optimization
for i in range(args.train_n):
  batch = mem.sample(BATCH_SIZE)
  metrics = learner.train(batch)
```



1. reward，state的时机

   - 请求调度(跟事件有关)

   - 动态迁移(action)

     -----5min------|(action，reward，state)------10min----

2. 时间序列建模

   初始化：node上面10个pod运行

   请求执行时间相同，资源量不同

   调度方法为纯轮询，

   历史5min每个pod上的请求量是近乎相等的，但请求资源量不同

   请求是串行执行的

   state是5min内node和pod的资源变化情况

   action是5min内相似请求模式下为了减少违约次数做的迁移操作

   reward根据action执行后5min内的相关指标计算

   

   原始数据：请求固定调度

   ----5min-----｜(action)---|(请求)-------｜(请求)-------｜action？reward

   资源考量建模：

   串行场景模拟：

   同个node上面的pod获取请求的间隔是近乎均匀的(round robin分配请求)

   以同个node上面5min内handle请求次数最多的pod的请求次数为横轴，不够的补0，把资源加和起来

   全部请求次数中违反资源量阈值的次数为资源考量指标

   并行场景模拟：

   请求分组，每一次handle申请的资源为累加的

   请求不是横轴

   性能考量建模：

   只考虑存在竞争，总资源超出node可分配资源的node上运行的请求个数(请求粒度)（最短作业优先，优先满足申请的资源最少的）TODO：update函数改

3. 尺寸一致：变长array->np.array 

   假设每个node上面最多分配100个pod