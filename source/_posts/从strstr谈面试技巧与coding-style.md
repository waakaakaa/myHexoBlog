title: '从strStr谈面试技巧与Coding Style'
date: 2015-10-15 12:37:11

---
从一道入门题说起：

	strStr
	/* Return the position of the first occurence of string target in string source, or -1 if target is not part of source.*/
	int strStr(String source, String target){
		//...
	}

<!-- more -->

常见错误：

1. 我知道一个算法叫KMP
2. 命名用s1, s2
3. for和if没有花括号

参考解法：

	class Solution{
		public int strStr(String source, String target){
			if (source == null || target == null){
				return -1;
			}
			
			int i,j;
			for (i = 0; i < source.length() - target.length() + 1; i++){
				for (j = 0; j < target.length(); j++){
					if (source.charAt(i + j) != target.charAt(j)){
						break;
					}
				}
				if (j == target.length()){
					return i;
				}
			}
			return -1;
		}	
	}

排列组合模板（Subsets）：

	private void subsetsHelper(ArrayList<Arraylist<Integer>> result, ArrayList<Integer> list, int[] nums, int pos){
		result.add(new ArrayList<Integer>(list));
		
		for (int i = pos; i < nums.length; i++){
			list.add(nums[i]);
			subsetsHelper(result, list, nums, i + 1);
			list.remove(list.size() - 1);
		}
	}

Unique Subsets：

	private void subsetsHelper(ArrayList<Arraylist<Integer>> result, ArrayList<Integer> list, int[] nums, int pos){
		result.add(new ArrayList<Integer>(list));
		
		for (int i = pos; i < nums.length; i++){
			if (i != pos && num[i] == nums[i - 1]){
				continue;
			}
			list.add(nums[i]);
			subsetsHelper(result, list, nums, i + 1);
			list.remove(list.size() - 1);
		}
	}