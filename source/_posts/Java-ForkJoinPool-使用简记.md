---
title: Java ForkJoinPool 使用简记
categories:
  - Java
tags:
  - Java
  - ForkJoinPool
  - 并发编程
date: 2018-06-30 10:56:18
---


Java 7提供了ForkJoinPool，实现了Fork/Join并行算法。fork操作将会启动一个新的并行Fork/Join子任务。join操作会一直等待直到所有的子任务都结束。
本文单记录一下ForkJoinPool的使用，以下是示例代码：
* 计算任务
```java
/**
* @since 1.7
*/
public class CalculatorComponent {
  private static final int PARALLELISM = Runtime.getRuntime().availableProcessors() +1 ;
  private static final ForkJoinPool forkJoinPool = new ForkJoinPool(PARALLELISM);

  List dataList = Lists.newArrayList(....); // 输入参数
  List<Object> subLits = forkJoinPool.invoke(new CalculateTask( dataList));

 class CalculateTask extends RecursiveTask<List<Object>> {

     private Deque<Object> dataList = null;

     public CalculateTask(List<Object> dataList) {
         this.dataList = new LinkedBlockingDeque<>();
         this.dataList.addAll(dataList);
     }

     @Override
     protected List<Object> compute() {

         if (dataList == null || dataList.size() == 0) {
             return Lists.newArrayList();
         }
         int size = dataList.size();
         if (size == 1) {
             //do something ,返回计算后的值
             return Lists.newArrayList(data);
         }
         List<Object> taskList = Lists.newArrayList();
         while (size > 0) {
             Object data = dataList.pop();
             CalculateTask left = new CalculateTask(Lists.newArrayList(data), seqId);
             taskList.add(left);
             size = dataList.size();
         }
         invokeAll(taskList);
         List<Object> resultList = Lists.newArrayList();
         for (CalculateTask task : taskList) {
             List<Object> tmp = task.join();
             resultList.addAll(tmp);
         }
         return resultList;
     }

 }
```


 > 需要注意的是编写 `RecursiveTask` 类 `compute()` 方法的时候需要注意要有判断分派任务终止的条件及处理。
 > 计算时是先分派完全部任务，之后才会执行最小计算单元。在实际运行过程中会发现 `if (size == 1){}` 代码块实际上是在 `invokeAll(taskList)` 之后执行的，
