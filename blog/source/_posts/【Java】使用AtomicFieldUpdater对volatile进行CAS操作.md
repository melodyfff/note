---
title: 【Java】使用AtomicFieldUpdater对volatile进行CAS操作
tags: [java]
date: 2020-7-02
---

# 使用AtomicFieldUpdater对volatile进行CAS操作

> {@link AtomicReferenceFieldUpdater} 基于反射，针对{@code volatile}修饰的指定字段fields进行原子更新操作
```java
/**
 * A reflection-based utility that enables atomic updates to
 * designated {@code volatile} reference fields of designated
 * classes. 
 **/
```
主要有以下几种
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater
- AtomicReferenceFieldUpdater

使用时机：

- 1.你想通过正常的引用使用volatile field，比如直接在类中调用this.variable,但是你也想时不时的使用一下CAS操作或者原子自增操作，那么你可以使用fieldUpdater
- 2.当你使用AtomicXXX的时候，其引用Atomic的对象有多个的时候，你可以使用fieldUpdater节约内存开销 , 因为{@link AtomicInteger}成员变量为 private volatile int value;而{@link AtomicReferenceFieldUpdater}没有成员变量,直接对引用操作。在操作很多原子对象时，可以达到节约开销的目的

## 使用实例
参考 `com.lmax.disruptor.SequenceGroups`原子并安全的添加和删除`sequence`

```java
/**
 * 1. 你想通过正常的引用使用volatile field，比如直接在类中调用this.variable,但是你也想时不时的使用一下CAS操作或者原子自增操作，那么你可以使用fieldUpdater
 * 2. 当你使用AtomicXXX的时候，其引用Atomic的对象有多个的时候，你可以使用fieldUpdater节约内存开销 , 因为{@link AtomicInteger}成员变量为 private volatile int value;
 * 而{@link AtomicReferenceFieldUpdater}没有成员变量,直接对引用操作
 * <p>
 * {@link AtomicReferenceFieldUpdater} 基于反射，针对{@code volatile}修饰的指定字段fields进行原子更新操作
 *
 * @author xinchen
 * @version 1.0
 * @date 02/07/2020 10:23
 * @see AtomicIntegerFieldUpdater
 * @see AtomicLongFieldUpdater
 * @see AtomicReferenceFieldUpdater
 */
public final class AtomicFieldUpdater {
    /**
     * 基于反射对volatile字段进行原子操作
     */
    private static final AtomicReferenceFieldUpdater<AtomicFieldUpdater, Value[]> VALUE_UPDATER =
            AtomicReferenceFieldUpdater.newUpdater(AtomicFieldUpdater.class, Value[].class, "values");
    private volatile Value[] values = new Value[]{new Value()};

    /**
     * safely and atomically added to
     *
     * @param hold        要修改的对象
     * @param updater     AtomicReferenceFieldUpdater
     * @param initVaule   设置要初始化的值
     * @param valuesToAdd 需要安全添加的对象
     */
    private void addValues(AtomicFieldUpdater hold,
                           AtomicReferenceFieldUpdater<AtomicFieldUpdater, Value[]> updater,
                           long initVaule,
                           Value... valuesToAdd) {

        Value[] currentValues;
        Value[] updateValues;
        do {
            // safely and atomically added to

            // 反射获取volatile Value[]
            currentValues = updater.get(hold);
            // 扩容
            updateValues = copyOf(currentValues, currentValues.length + valuesToAdd.length);

            // 初始化扩容对象，并赋予初始值
            int index = currentValues.length;
            for (Value value : valuesToAdd) {
                value.value = initVaule;
                updateValues[index++] = value;
            }

            // CAS safely and atomically added to
        } while (!updater.compareAndSet(hold, currentValues, updateValues));

    }

    /**
     *
     * @param hold 要修改的对象
     * @param updater AtomicReferenceFieldUpdater
     * @param valuesToRemove 需要移除的对象
     * @return 移除结果
     */
    private boolean removeValues(AtomicFieldUpdater hold,
                              AtomicReferenceFieldUpdater<AtomicFieldUpdater, Value[]> updater,
                              Value valuesToRemove) {
        int numToRemove;
        Value[] oldValues;
        Value[] newValues;
        do {

            oldValues = updater.get(hold);

            numToRemove = countMatching(oldValues, valuesToRemove);

            if (0 == numToRemove)
            {
                break;
            }

            final int oldSize = oldValues.length;
            newValues = new Value[oldSize - numToRemove];
            for (int i = 0,pos = 0; i < oldSize; i++) {
                Value testValue = oldValues[i];
                // 填充newValues，忽略valuesToRemove
                if (valuesToRemove != testValue){
                    newValues[pos++] = testValue;
                }
            }

        } while (!updater.compareAndSet(hold, oldValues, newValues));

        return numToRemove != 0;
    }

    private static int countMatching(Value[] values,Value toMatch){
        int numToRemove = 0;
        for (Value value : values) {
            // Specifically uses identity
            if (value==toMatch){
                numToRemove++;
            }
        }
        return numToRemove;
    }

    static class Value {
        volatile long value = 1L;
    }


    public static void main(String[] args) {
        // 创建
        AtomicFieldUpdater atomicFieldUpdater = new AtomicFieldUpdater();

        // safely and atomically added to
        Value value = new Value();
        atomicFieldUpdater.addValues(atomicFieldUpdater, AtomicFieldUpdater.VALUE_UPDATER, 10, value);

        System.out.println();


        atomicFieldUpdater.removeValues(atomicFieldUpdater,AtomicFieldUpdater.VALUE_UPDATER,value);

        System.out.println();
    }
}
```