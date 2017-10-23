### Majority Element II          

Given an integer array of size *n*, find all elements that appear more than `⌊ n/3 ⌋` times. The algorithm should run in linear time and in O(1) space.

* 从题意可知，超过n/3的数可能有0、1或2个

* 先假设有两个，为num1，num2,则数列可分为num1,num2,和非num1和num2这三类，我们容易的到这样一个事实，这三类数字同时减少一个，不会改变最终结果，总数大于n/3的数字总数仍大于n/3

* 在这道题中，如果我们同时减去的三个互补相同的数字可能有以下几种情况：

  *  减去三个非num1和num2的数字
  * 减去一个num1两个非num1和num2的数字
  * 减去一个num2两个非num1和num2的数字
  * 减去一个num1一个num2和一个非num1和num2的数字

  以上几种情况都不会影响最终结果

* 最后要解决的就是怎么保证减去的都是互不相同的数字，具体实现看代码，感觉就像一个栈一样，有两个栈分别为num1、num2，如果当前数等于其中一个，则把数字放入对应的栈，否则两个栈的栈顶都出栈，如果栈空了就把当前数字放入栈内，最后栈内的数字一定包含大于n/3的数字

```
class Solution {
    public List<Integer> majorityElement(int[] nums) {
        int total1=0,total2=0,num1=9,num2=99;
        for (int i=0;i<nums.length;i++){
            if (nums[i]==num1){
                total1++;
                continue;
            }
            if (nums[i]==num2){
                total2++;
                continue;
            }
            if (total1==0){
                num1=nums[i];
                total1=1;
                continue;
            }
            if (total2==0){
                num2=nums[i];
                total2=1;
                continue;
            }
            total1--;
            total2--;
        }
        total1=0;
        total2=0;
        for (int i=0;i<nums.length;i++){   
            if (num1==nums[i]) total1++;
            if (num2==nums[i]) total2++;
        }
        List<Integer> list=new ArrayList();
        if (total1>nums.length/3) list.add(num1);
        if (total2>nums.length/3) list.add(num2);
        return list;
    }
}
```

