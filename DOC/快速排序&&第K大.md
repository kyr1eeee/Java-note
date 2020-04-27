##  快速排序

#### 挖坑法

```java
void Sort(int[] nums){
    quickSort(nums,0,nums.length-1);
}
void quickSort(int[] nums,int low,int hi){
    if(low>=hi) return;
    int start=lo;
    int end=hi;
    int key=nums[start];
    while(start<end){
        while(start<end&&nums[end]>=key) end--;
        nums[start]=nums[end];
        while(start<end&&nums[start]<=key) start++;
        nums[end]=nums[start];
    }
    nums[start]=key;
    quickSort(nums,low,start-1);
    quickSort(nums,start+1,hi);
}
```

#### 左右指针

```java
void quickSort(int[] nums,int low,int hi){
    if(low>=hi) return;
    int start=lo;
    int end=hi;
    int key=nums[start];
    while(start<end){
        while(start<end&&nums[end]>=key) end--;
        //nums[start]=nums[end];
        while(start<end&&nums[start]<=key) start++;
        //nums[end]=nums[start];
        swap(nums,start,end);
    }
    nums[start]=key;
    quickSort(nums,low,start-1);
    quickSort(nums,start+1,hi);
}
```

#### 前后指针

```java
void quickSort(int[] nums,int low,int hi){
    if(low>=hi) return;
    int cur=low;
    //int end=hi;
    int key=nums[hi];
    int pre=cur-1;
    while(cur<hi){
        if(nums[cur]<=key){
            pre++;
            if(pre!=cur){
                int temp=nums[pre];
                nums[pre]=nums[cur];
                nums[cur]=temp;
            }
        }
        cur++;
    }
    //int temp=nums[++pre];
    nums[hi]=nums[++pre];
    nums[pre]=key;
    quickSort(nums,low,start-1);
    quickSort(nums,start+1,hi);
}
```

(注意：前后指针法由于只有往前移动，因此适用于像链表这种数据结构)

### 拓展

#### 寻找数组中第k大的数

```java
//通过快速排序
int findK(int[] nums,int k){
    int len=nums.length;
    return parttion(nums,0,len-1,k);
}
int parttion(int[] nums,int lo,int hi,int k){
    	int start=lo;
        int end=hi;
        int key=nums[start];
        while(start<end){
            while(start<end&&nums[end]>=key) end--;
            nums[start]=nums[end];
            while(start<end&&nums[start]<=key) start++;
            nums[end]=nums[start];
        }
        nums[start]=key;
        if(start==k-1) return key;
        else if(start<k-1) return parttion(nums,start+1,hi,k);
        else return parttion(nums,lo,start-1,k);
    
}
//也可以冒泡排序
int findK(int[] nums,int k){
    boolean notsorted=true;
    int i=len-1;
    for(int i=0;i<k&&notSorted;i++){
        flag=false;
        for(int j=len-1;j>i;j--){
            //把大的数往前移
            if(nums[j-1]<nums[j]){
                swap();
                flag=true;
            }
        }
    }
    return nums[k-1];
}
//基于选择排序
int findK(int[] nums,int k){
    for(int i=0;i<k;i++){
        for(int j=i+1;j<len;j++){
            if(nums[j]>nums[i])
                swap();
        }
    }
    return nums[k-1];
}
//借助外部数据结构最小堆，可以实现海量数据的查找
int findK(int[] nums,int k){
    PriorityQueue<Integer> pq=new PriorityQueue<Integer>();
    for(int i=0;i<nums.length;i++){
        pq.add(nums[i]);
        if(pq.size()>k)
            pq.poll();
        
    }
    return pq.peek();
}
//更加复杂的可以采取各种排序算法，对数组排序后再取相应数。
```

