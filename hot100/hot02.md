1.字母异位词分组
[[https://leetcode.cn/problems/group-anagrams/description/?envType=study-plan-v2&envId=top-100-liked]]
时间复杂度O(n * k log k),空间复杂度：`O(n * k)`

```java
class Solution {

    public List<List<String>> groupAnagrams(String[] strs) {

        Map<String,List<String>> map = new HashMap<>();

        for (String s : strs){

            // 把字符串转化为字符数组

            char[] arr = s.toCharArray();

            // 对字符串进行排序

            Arrays.sort(arr);

            // 重新转成字符串并记录为key

            String key = new String(arr);

            // 如果该字符串没有在hashmap中记录,就把这个字符串放入hashmap中的一个新的key中

            if (!map.containsKey(key)){

                map.put(key,new ArrayList<>());

            }

            // 如果存在就把字符串添加进value的list中

            map.get(key).add(s);

        }

        return new ArrayList<>(map.values());

    }

}
```



第二种方法不排序
```java
import java.util.*;

class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();

        for (String s : strs) {
            // 1) 统计 26 个字母频次
            int[] count = new int[26];
            for (int i = 0; i < s.length(); i++) {
                char c = s.charAt(i);
                count[c - 'a']++;
            }

            // 2) 把频次数组拼成字符串 key
            //    用分隔符避免歧义，比如 "#1#0#0..."
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < 26; i++) {
                sb.append('#');
                sb.append(count[i]);
            }
            String key = sb.toString();

            // 3) 常规 map 写法（不用 computeIfAbsent）
            if (!map.containsKey(key)) {
                map.put(key, new ArrayList<>());
            }
            map.get(key).add(s);
        }

        // 4) 返回所有分组
        return new ArrayList<>(map.values());
    }
}

```