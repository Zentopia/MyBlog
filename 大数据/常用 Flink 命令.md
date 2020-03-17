# 常用 Flink 命令

## Yarn

kill yarn application
`yarn application -kill [application id]`

Flink run job mode
`./bin/flink run -m yarn-cluster ./path/to/job.jar`

## Flink

### Savepoint

take a savepoint
```bash
./bin/flink savepoint <jobId> 􏰶savepointPath􏰷
```

disposing a savepoint
```bash

```

delete savepoint
```bash
./bin/flink savepoint -d <savepointPath>
```


### CANCELING AN APPLICATION
```bash
./bin/flink cancel <jobId>
```

In order to take a savepoint before canceling a running application add the 􏰧􏰘 flag to the 􏰖􏰂􏰍􏰖􏰅􏰐 command:
```bash
./bin/flink cancel -s 􏰶savepointPath􏰷 <jobId>
```

STARTING AN APPLICATION FROM A SAVEPOINT
```bash
./bin/flink run -s <savepointPath> 􏰶options􏰷 <jobJar> 􏰶arguments􏰷
```