# Distributed Database

**Primary key** 主键

但是有的时候主键可能是**Composite/Compound**的 (比如它涉及多个columns), 一般compound key的第一个部分就是**partition key**，第二个部分是**clustering key**。它们也能涉及多个column,比如 primary key ((key_part_one, key_part_two), k_clust_one, k_clust_two,...).

**Partition key** is responsible for data distribution across nodes

**Clustering key** is reponsible for data sorting within the partition



