
### 作业描述

#### 使用 Java API 操作 HBase

建表，实现插入数据，删除数据，查询等功能。建立一个如下所示的表：

- 表名：$your_name:student
- 空白处自行填写, 姓名学号一律填写真实姓名和学号

![img](https://typora-mac-alpha.oss-cn-shanghai.aliyuncs.com/img/1647657242794-8ddc75b3-1105-4269-9ab6-90602b36d36d.png)



### 实战

部署hbase docker 环境

```shell
#拉取镜像
docker pull harisekhon/hbase
#启动容器
docker run -d -p 2181:2181 -p 8080:8080 -p 8085:8085 -p 9090:9090 -p 9095:9095 -p 16000:16000 -p 16010:16010 -p 16201:16201 -p 16301:16301 -p 16030:16030 -p 16020:16020 --name jikeshijian_hbase harisekhon/hbase
```

配置系统hosts，追加

127.0.0.1 容器ID    



进入容器

进入habase shell

创建namespace

```shell
create_namespace 'afa'
```



代码操作Hbase

```java
package com.afa.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HbaseDemo {
    public static void main(String[] args) throws IOException {
        System.out.println("极客时间-作业2 阿发-G20220735020087");

        // 建立连接
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "127.0.0.1");
        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        configuration.set("hbase.master", "127.0.0.1:60000");
        Connection conn = ConnectionFactory.createConnection(configuration);
        Admin admin = conn.getAdmin();

        TableName tableName = TableName.valueOf("afa:student");
        String colFamily1 = "info";
        String colFamily2 = "score";

        // 建表
        if (admin.tableExists(tableName)) {
            System.out.println("Table already exists");
        } else {
            HTableDescriptor hTableDescriptor = new HTableDescriptor(tableName);
            HColumnDescriptor hColumnDescriptor1 = new HColumnDescriptor(colFamily1);
            HColumnDescriptor hColumnDescriptor2 = new HColumnDescriptor(colFamily2);
            hTableDescriptor.addFamily(hColumnDescriptor1);
            hTableDescriptor.addFamily(hColumnDescriptor2);
            admin.createTable(hTableDescriptor);
            System.out.println("Table create successful");
        }

        // 插入数据
        String rowKey = "afa";
        Put put = new Put(Bytes.toBytes(rowKey)); // row key
        put.addColumn(Bytes.toBytes(colFamily1), Bytes.toBytes("student_id"), Bytes.toBytes("G20220735020087")); // col1
        put.addColumn(Bytes.toBytes(colFamily1), Bytes.toBytes("class"), Bytes.toBytes(String.valueOf(1))); // col2
        put.addColumn(Bytes.toBytes(colFamily2), Bytes.toBytes("understanding"), Bytes.toBytes(String.valueOf(99))); // col1
        put.addColumn(Bytes.toBytes(colFamily2), Bytes.toBytes("programing"), Bytes.toBytes(String.valueOf(99))); // col2
        conn.getTable(tableName).put(put);
        System.out.println("Data insert success");

        // 查看数据
        Get get = new Get(Bytes.toBytes(rowKey));
        if (!get.isCheckExistenceOnly()) {
            Result result = conn.getTable(tableName).get(get);
            for (Cell cell : result.rawCells()) {
                String colName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                System.out.println("Data get success, colName: " + colName + ", value: " + value);
            }
        }

        // 删除数据
        Delete delete = new Delete(Bytes.toBytes(rowKey));      // 指定rowKey
        conn.getTable(tableName).delete(delete);
        System.out.println("Delete Success");

        // 删除表
        if (admin.tableExists(tableName)) {
            admin.disableTable(tableName);
            admin.deleteTable(tableName);
            System.out.println("Table Delete Successful");
        } else {
            System.out.println("Table does not exist!");
        }
    }
}
```

程序输出结果

![img](https://cdn.nlark.com/yuque/0/2022/png/2981563/1647660099990-f5118a16-d9a2-4992-bc3f-bfd637fdde2b.png)





表的信息，两个列簇 info、与score

![img](https://cdn.nlark.com/yuque/0/2022/png/2981563/1647659718327-6bd7b092-4a7d-44b0-8259-ecefc63ed2b1.png)



插入的数据

![img](https://cdn.nlark.com/yuque/0/2022/png/2981563/1647659883953-4ff93ab6-22f0-4694-87e4-090e3ac879de.png)
