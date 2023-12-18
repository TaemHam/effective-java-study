# 아이템 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

`java.util.concurrent` 패키지가 없었던 과거에는 멀티스레드 프로그래밍을 위해 `java.lang.Thread`나 `java.lang.Runnable`을 사용했는데,<br>
안전 실패나 응답 불가 같은 동시성 문제를 방지하기 위한 코드를 직접 삽입해야했고, 때문에 작업 코드는 간단하더라도 그 길이가 수십 줄은 늘어났다.

> **📌 참고**<br>
> 
> `Thread`와 `Runnable`의 단점
> * 지나치게 저수준의 API(스레드의 생성)에 의존
> * 값의 반환이 불가능
> * 매번 스레드 생성과 종료하는 오버헤드가 발생
> * 스레드들의 관리가 어려움

하지만 `java.util.concurrent` 패키지가 등장하고 나서는, 이 문제들을 해결하기 위해 긴 코드를 작성할 필요가 없어졌다.

이 concurrent 패키지 중에서도 멀티스레드 관리를 도와주는 `실행자 프레임워크`에 대해 알아보자.

## 실행자 프레임워크(Executor Framework)

실행자 프레임워크는 크게 세 가지로 구분지을 수 있다.

1. Executor : **스레드 풀의 구현을 위한 인터페이스**. 작업 등록과 작업 실행 중에서 **작업 실행만을 책임짐**.

2. ExecutorService : **작업(Runnable, Callable) 등록을 위한 인터페이스**. 라이프사이클 관리와 비동기 작업을 위한 기능들을 제공.

3. Executors : 위의 스레드 풀 인터페이스의 **구현체들을 정적 팩터리 메서드로 제공**.

실행자 프레임워크는 다음과 같이 사용할 수 있다.

```JAVA
// 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 작업 큐 실행
exec.execute(runnable);

// 작업 큐 삭제
exec.shutdown();
```

<details>
<summary>`ExecutorService` 구현체의 `.execute()` 메서드 내부 코드</summary>

```JAVA
public class ThreadPoolExecutor extends AbstractExecutorService {

    ...

    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
}
```
</details>

### 실행자 서비스의 기능

실행자 서비스는 크게 라이프사이클 관리를 위한 기능들과 비동기 작업을 위한 기능들로 나눌 수 있다.

* 라이프사이클 관리를 위한 메서드

  * `.shutdown()` <br>
    새로운 작업들을 더 이상 받아들이지 않음 <br>
    호출 전에 제출된 작업들은 그대로 실행이 끝나고 종료됨(Graceful Shutdown)

  * `.shutdownNow()` <br>
    shutdown 기능에 더해 이미 제출된 작업들을 인터럽트 시킴 <br>
    실행을 위해 대기중인 작업 목록(`List<Runnable>`)을 반환함

  * `.isShutdown()` <br>
    Executor의 shutdown 여부를 반환함

  * `.isTerminated()` <br>
    shutdown 실행 후 모든 작업의 종료 여부를 반환함

  * `.awaitTermination()` <br>
    shutdown 실행 후, 지정한 시간 동안 모든 작업이 종료될 때 까지 대기함 <br>
    지정한 시간 내에 모든 작업이 종료되었는지 여부를 반환함

* 비동기 작업을 위한 기능들

  * `.submit()` <br>
    실행할 작업들을 추가하고, 작업의 상태와 결과를 포함하는 Future를 반환함 <br>
    Future의 get을 호출하면 성공적으로 작업이 완료된 후 결과를 얻을 수 있음

  * `.invokeAll()` <br>
    모든 결과가 나올 때 까지 대기하는 블로킹 방식의 요청 <br>
    동시에 주어진 작업들을 모두 실행하고, 전부 끝나면 각각의 상태와 결과를 갖는 `List<Future>`을 반환함

  * `.invokeAny()` <br>
    가장 빨리 실행된 결과가 나올 때 까지 대기하는 블로킹 방식의 요청 <br>
    동시에 주어진 작업들을 모두 실행하고, 가장 빨리 완료된 하나의 결과를 Future로 반환 받음

### 스레드 풀의 종류

#### `ThreadPoolExecutor`
커스텀할 수 있는 스레드 풀 <br>
스레드 풀 동작을 결정하는 거의 모든 속성을 설정 가능

#### `Executors.newCachedThreadPool()`
요청받은 태스크를 큐에 쌓지 않고 바로 처리하며, <br>
사용 가능한 스레드가 없다면 즉시 스레드를 새로 생성해 처리 <br>
가벼운 프로그램을 실행하는 서버에 적합

#### `Executors.newFixedThreadPool()`
스레드 개수를 고정하는 스레드풀 <br>
무거운 프로덕션 서버에 적합

#### `ForkJoinPool`

작은 하위 태스크로 나눌 수 있는 `ForkJoinTask`를 실행하는 스레드풀 <br>
최대한의 CPU를 사용해서, 높은 처리량과 낮은 지연시간을 달성 <br>
병렬 스트림을 이용하면 어렵지 않게 사용 가능

<img src="https://velog.velcdn.com/images/yhlee9753/post/a4c5f878-505a-4c59-b040-31646d4dcdb9/image.png" alt="포크조인" style="background-color: white; max-width: 720px; width: 100%; height: auto; margin: 0 auto;">