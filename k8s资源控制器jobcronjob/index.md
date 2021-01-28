# k8s资源控制器JobCronJob

## Job

Job 负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个 Pod 成功结束<br>

**特殊说明：**

- spec.template 格式同 Pod
- RestartPolicy 仅支持 Never 或 OnFailure
- 单个 Pod 时，默认 Pod 成功运行后 Job 即结束
- `.spec.completions`：标志 Job 结束需要成功运行的 Pod 个数，默认为 1
- `.spec.parallelism`：标志并运行的 Pod 的个数，默认为 1
- `.spec.activeDeadlineSecond`：标志失败 Pod 的重试最大时间，超过这个时间不会继续重试

**For Expamle**

```yaml
kind: Job
metadata:
  name: pi
spec:
  template:
    metadata:
      name: pi
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```
## CronJob

**_CronJob_** 管理基于时间的 Job，即：

- 在给定时间点只运行一次
- 周期性地在个体定时间点运行

典型用法如下所示：

- 在给定的时间点调度 Job 运行
- 创建周期性运行的 Job，例如：数据库的备份、发送邮件

**特殊说明：**

- `.spec.schedule`：调度，必须字段，指定任务运行周期，格式同 linux Cron。
- `.spec.jobTemplate`：Job 模板，必须字段，指定需要运行的任务，格式同 Job。
- `.spec.startingDeadlineSeconds`：启动 Job 的期限（秒级别），该字段是可选的，如果因为任何原因而错过卢被调度的时间，那么错过执行时间的 Job 将被认为是失败的，如果没有指定，则没有期限。
- `.spec.concurrencyPolicy`：并发策略，该字段也是可选的，它指定卢如何处理被 CronJob 创建的 Job 的并发执行，只允许下面策略中的一种。
  - `Allow`（默认）：允许并发运行 Job。
  - `Forbid`：禁止并发运行，如果前一个还没有完成，则直接跳过下一个。
  - `Replace`：取消当前正在运行的 Job，用一个新的来替换。<br>
    注意：当前策略只能应用于同一个 CronJob 创建的 Job，如何存在多个 CronJob，他们创建的 Job 之间总是并发运行的。
- `.spec.suspend`：挂起，该字段也是可选的，如果设为`true`，后续所有执行都会被挂起，它对已经开始执行的 Job 不起作用，默认值为`false`。
- `.spec.successfulJobHistoryLimit`和`.spec.failedJobsHistoryLimit`：历史限制，可选字段，它们指定了可以保留多少完成和失败的 Job，默认情况下，它们分别设置 3 和 1，设置限制的值为 0，相关类型的 Job 完成后将不会被保留。

**For Expamle**

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              args:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

