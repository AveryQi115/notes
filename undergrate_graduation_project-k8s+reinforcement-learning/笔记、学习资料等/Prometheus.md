- 文档：https://prometheus.io/docs/prometheus/latest/getting_started/

# 基本操作

1. 启动待监控的容器或服务
2. 配置prometheus config file，启动prometheus
3. 指定ip+metrics可看到metrics输出，graph可以看到图表类输出，query language可以查询聚合后的打点

- 下载docker镜像，启动prometheus容器

  ![](/Users/a77/hw/毕业实训/img/pro_panel.jpg)

- Localhost:port/metrics输出监控得到的metrics

  ![](/Users/a77/hw/毕业实训/img/pro_metrics.jpg)

- 查看graph示例

  ![](/Users/a77/hw/毕业实训/img/pro_graph.jpg)

  