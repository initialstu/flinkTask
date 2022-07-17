# flinkTask

## 题目：flink流式处理
report(transactions).executeInsert(“spend_report”);将 transactions 表经过 report 函数处理后写入到 spend_report 表。

每分钟（或小时）计算在五分钟（或小时）内每个账号的平均交易金额（滑动窗口）？使用分钟还是小时作为单位均可。

## 运行环境
电脑上需要安装如下环境：Java 8 或者 Java 11、Maven、Git、Docker

## 代码环境
配置文件位于 flink-playgrounds 仓库中，首先检出该仓库并构建 Docker 镜像：
> git clone  https://github.com/apache/flink-playgrounds.git

## 修改代码
* 找到 SpendReport.java
* 实现方法 public static Table report(Table transactions)
```java
public static Table report(Table transactions) {
    return transactions
            .window(Slide.over(lit(5).minutes())
                    .every(lit(1).minutes())
                    .on($("transaction_time"))
                    .as("log_ts"))
            .groupBy($("account_id"), $("log_ts"))
            .select($("account_id"),
                    $("log_ts").start().as("log_ts"),
                    $("amount").avg().as("amount"));
}
```

## 构建docker镜像
* 启动docker服务（windows打开docker desktop即可）
* 进入目录
> cd flink-playgrounds/table-walkthrough
* 编译打包代码（windows执行踩坑，代码中换行符如果是\r\n，则在build时会报错，主要是执行到mvn spotless:check代码格式化检查时会报错。需要把代码中换行符改为\n，然后build可以成功）
> docker-compose build
* 启动服务环境
> docker-compose up -d
* 查看启动服务
> docker ps

## 查看和验证
* Flink WebUI
> http://localhost:8082
* 查看日志
> docker-compose logs -f jobmanager
> 
> docker-compose logs -f taskmanager
* 查看mysql表数据更新
> docker-compose exec mysql mysql -Dsql-demo -usql-demo -pdemo-sql
> 
> mysql> use sql-demo;
> 
> mysql> select count(*) from spend_report;
* Grafana查看监控
> http://localhost:3000/d/FOe0PbmGk/walkthrough?viewPanel=2&orgId=1&refresh=5s

## 结束
* 关闭服务环境
> docker-compose down -v
