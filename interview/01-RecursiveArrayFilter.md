# 递归过滤数组元素
相信面试过程中各位都会遇到对递归思想的考察。这里记录一道小面试题，可以作为面试的前菜。

## 问题

给定一个整型数组origin，要求使用递归的方式过滤掉其中不满足指定条件(如大于3)的元素。

## 分析

对于递归类的问题，有两个点需要重点考虑：
- 问题分解
- 递归终止条件

具体到这个问题，首先我们引入两个变量方便说明问题。start,对origin进行过滤的起始元素的索引；length:origin数组的长度。

首先将原数组分割为两个数组，分别为origin[start] 和 origin[start+1] origin[start+2]
origin[start+3] ... origin[length-1]。然后对前者进行条件判断，对后者进行递归调用，最后把结果合并。

那递归的终止条件是什么？上述的问题分解过程不可能一直持续下去。当分解到只需要从origin的最后一个元素开始过滤的时候，就不需要进行递归调用了。


    public class ArrayTest {
    
        @Test
        public void testArrayFilter(){
            int[] origin = new int[]{0, 1, 2, 3, 4, 5, 6};
            int[] after = doFilter(origin, 0);
            System.out.println(after);
        }
    
        private int[] doFilter(int[] origin, int start){
    
            //退化成最简单的case:从原数组的最后一个元素筛选。这也是递归终止条件
            if(start == origin.length-1){
                int[] result = new int[1];
                result[0] = origin[start];
                return result;
            }
    
            int[] result;
            int[] subResult = doFilter(origin, start+1);
    
            if(filter(origin[start])){
                result = new int[subResult.length+1];
                result[0] = origin[start];
                for(int i=0;i<subResult.length;i++){
                    result[i+1] = subResult[i];
                }
                return result;
            }else{
                return subResult;
            }
        }
    
        private boolean filter(int m){
            if(m>3){
                return true;
            }
            return false;
        }
    }