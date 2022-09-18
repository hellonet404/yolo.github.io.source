---
title: C#中的字典
tags:
  - Dictionary
categories:
  - 基础数据接口
date: 2022-09-18 12:30:45
---

##### Diciotnary

```c#
public class Dictionary<Tkey, TValue>
    {
        private int[] buckets;
        private Entry[] entries;
        private int count;
        private int version;
        private int freeList;
        private int freeCount;
        private IEqualityComparer<Tkey> comparer;


        public Dictionary()
        {
            this.comparer = EqualityComparer<Tkey>.Default;
        }

        private struct Entry
        {
            public int hashCode;//计算的hashCode
            public int next;//指向下一个entry的指针
            public Tkey key;// key
            public TValue value; // value
        }

        public void Insert(Tkey key, TValue value)
        {

            if (buckets == null) Initialize(6); //初始化
            int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;//计算hashCode
            int targetBucket = hashCode % buckets.Length;//计算碰撞桶的坐标

            // 查看桶的值，循环
            for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next) 
            {
                //判断插入的key 和 value 都相同就覆盖原来的值
                if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))
                {
                    entries[i].value = value;
                    version++;
                    return;
                }

               
            }
            int index;
            //有多少个被删除的Entry，有多少个空闲的位置
            if (freeCount > 0) 
            {
                index = freeList;
                freeList = entries[index].next;
                freeCount--;
            }
            else
            {
                //count 当前entries的index位置，就是到达了entry最后的一个位置 空间不够了
                if (count == entries.Length)
                {
                    // 扩容操作
                    Resize(17, false);// 7是通过内部函数重新找刀合适Capcity
                    targetBucket = hashCode % buckets.Length;
                }
                index = count;
                count++;
            }

            entries[index].hashCode = hashCode;
            //将当前entry的next指向上一个存储的entry的下标
            entries[index].next = buckets[targetBucket];
            entries[index].key = key;
            entries[index].value = value; 
            //将桶的值赋值为当前entry的下标。说白了就是让当前桶槽指向当前的entry
            buckets[targetBucket] = index; 
            version++;
        }

        private void Initialize(int capacity)
        {
            // 初始化桶的长度，源码是找到 离 capacity 最小的素数（也就是质数）
            buckets = new int[7];
            // 桶里面的值全设置为-1
            for (int i = 0; i < buckets.Length; i++) buckets[i] = -1; 
            //初始化entry的长度
            entries = new Entry[7];
            // 被删除Entry在entries中的下标index，这个位置是空闲的
            freeList = -1;
        }

        public bool TryGetValue(Tkey key, out TValue value)
        {
            int i = FindEntry(key);
            if (i >= 0)
            {
                value = entries[i].value;
                return true;
            }
            value = default(TValue);
            return false;
        }

        private int FindEntry(Tkey key)
        {
            if (buckets != null)
            {
                //计算hashCode
                int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF; 
                //根据碰撞刀桶的值进行循环
                for (int i = buckets[hashCode % buckets.Length]; i >= 0; i = entries[i].next)
                {
                    //如果取值的key和hashcode和entry的key还有hashCode都相同，返回entry的索引
                    if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) return i; 
                }
            }
            return -1;
        }

        public bool Remove(Tkey key)
        {
            if (buckets != null)
            {
                int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
                int bucket = hashCode % buckets.Length;
                int last = -1;
                for (int i = buckets[bucket]; i >= 0; last = i, i = entries[i].next)
                {
                    if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))
                    {
                        if (last < 0)
                        {
                            //当删除最后一个entry时，当前槽的值设置为-1
                            buckets[bucket] = entries[i].next; 
                        }
                        else
                        {
                            //将要删除的entry指向的下标赋值给下一个entry的下
                            entries[last].next = entries[i].next; 标
                        }
                        entries[i].hashCode = -1;
                        entries[i].next = freeList;
                        entries[i].key = default(Tkey);
                        entries[i].value = default(TValue);
                        freeList = i;
                        freeCount++;
                        version++;
                        return true;
                    }
                }
            }
            return false;
        }

        /// <summary>
        /// 扩容
        /// </summary>
        /// <param name="newSize"></param>
        /// <param name="forceNewHashCodes"></param>
        private void Resize(int newSize, bool forceNewHashCodes)
        {
            Contract.Assert(newSize >= entries.Length);
            int[] newBuckets = new int[newSize];
            //1. 初始化新的buckets
            for (int i = 0; i < newBuckets.Length; i++) newBuckets[i] = -1;
            //2. 初始化新的entrys
            Entry[] newEntries = new Entry[newSize]; 
            //3. 将原来的entrys 复制给新的 entry
            Array.Copy(entries, 0, newEntries, 0, count);
            //4. 是否重新计算hashCode
            if (forceNewHashCodes) 
            {
                for (int i = 0; i < count; i++)
                {
                    if (newEntries[i].hashCode != -1)
                    {
                        newEntries[i].hashCode = (comparer.GetHashCode(newEntries[i].key) & 0x7FFFFFFF);
                    }
                }
            }
            for (int i = 0; i < count; i++)
            {
                if (newEntries[i].hashCode >= 0)
                {
                    //5. 重新计算每个entry对应的桶槽
                    int bucket = newEntries[i].hashCode % newSize;
                    newEntries[i].next = newBuckets[bucket];
                    newBuckets[bucket] = i;
                }
            }
            buckets = newBuckets;
            entries = newEntries;
        }
    }
```

