# Upgrade Elasticsearch 7.17 to 8.x

Make sure there is a current snapshot taken before beginning the upgrade.

Use Upgrade Assistant to identify and resolve any issues and reindex indices created prior to Elasticsearch 7.

![1_Elasticsearch deprecation issues](https://user-images.githubusercontent.com/104564793/182584599-185f0303-708f-40d0-9947-17f8b56ef97e.png)

Various settings in `elasticsearch.yml` are deprecrated and must be updated - Upgrade Assistant provides guidance on what should be done
![6_node master is deprecated msg](https://user-images.githubusercontent.com/104564793/182584818-165a6e27-3856-423f-b9f9-e93f835bcd9f.png)
![10_master-1 node role master ingest](https://user-images.githubusercontent.com/104564793/182586169-b935ffc5-e2c2-4821-ba52-358ca12eeab8.png)

Indices created prior to version 7 must be reindexed - accept Upgrade Assistant's changes to complete reindexing for each index
![2_Reindex dialog](https://user-images.githubusercontent.com/104564793/182584696-e3b3c62b-a436-47fa-a09b-cfd721a24727.png)
![3_Reindex dialog2](https://user-images.githubusercontent.com/104564793/182585075-d48b47f5-309e-4cb8-b854-ccb195b57eb0.png)
![4_Reindex of one index complete](https://user-images.githubusercontent.com/104564793/182586038-701be68f-095c-40ae-acc4-7eafbce4d6fa.png)
