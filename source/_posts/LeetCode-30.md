---
title: LeetCode：30.串联所有单词的子串
date: 2024-03-29 22:45:00
category: 代码片段
tags: LeetCode
cover: img/leetcode.png
---
给定一个字符串 `s` 和一个字符串数组 `words`。 `words` 中所有字符串 长度相同。

`s` 中的 串联子串 是指一个包含  `words` 中所有字符串以任意顺序排列连接起来的子串。

- 例如，如果 `words = ["ab","cd","ef"]`， 那么 `"abcdef"`， `"abefcd"`，`"cdabef"`， `"cdefab"`，`"efabcd"`， 和 `"efcdab"` 都是串联子串。 `"acdbef"` 不是串联子串，因为他不是任何 `words` 排列的连接。

返回所有串联子串在 `s` 中的开始索引。你可以以 任意顺序 返回答案。

示例 1：
> 输入：s = "barfoothefoobarman", words = ["foo","bar"]
> 
> 输出：[0,9]
> 
> 解释：因为 words.length == 2 同时 words[i].length == 3，连接的子字符串的长度必须为 6。
> 
> 子串 "barfoo" 开始位置是 0。它是 words 中以 ["bar","foo"] 顺序排列的连接。
> 
> 子串 "foobar" 开始位置是 9。它是 words 中以 ["foo","bar"] 顺序排列的连接。
> 
> 输出顺序无关紧要。返回 [9,0] 也是可以的。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    public static List<Integer> findSubstring(String s, String[] words) {
        List<Integer> res = new ArrayList<Integer>();
        int wordNum = words.length, wordLen = words[0].length(), sLen = s.length();

        for (int i = 0; i < wordLen; i++) {
            if (i + wordNum * wordLen > sLen) {
                break;
            }
            // 初始化一个 hashMap
            Map<String, Integer> diff = new HashMap<>();

            // 从 hashMap 里移除目标 word，可以理解为：为每个 word 挖一个坑
            for (String word : words) {
                diff.put(word, diff.getOrDefault(word, 0) - 1);
            }

            // 往 hashMap 里添加 word，可以理解为：用 word 进行填坑
            for (int j = 0; j < wordNum; j++) {
                String word = s.substring(i + j * wordLen, i + (j + 1) * wordLen);
                diff.put(word, diff.getOrDefault(word, 0) + 1);
                // 如果刚好填了对应的坑，就把痕迹抹除
                if (diff.get(word) == 0) {
                    diff.remove(word);
                }
            }

            for (int begin = i; begin < sLen - wordNum * wordLen + 1; begin += wordLen) {
                if (begin != i) {
                    // 往 diff 里添加一个新的 word
                    String wordAdd = s.substring(begin + (wordNum - 1) * wordLen, begin + wordNum * wordLen);
                    diff.put(wordAdd, diff.getOrDefault(wordAdd, 0) + 1);
                    // 若 value 大于 0 说明: 该 word 没有对应的坑
                    if (diff.get(wordAdd) == 0) {
                        diff.remove(wordAdd);
                    }
                    // 从 diff 里移除一个旧的 word
                    String wordSub = s.substring(begin - wordLen, begin);
                    // 若 value 小于 0 说明: 坑没有被填平
                    diff.put(wordSub, diff.getOrDefault(wordSub, 0) - 1);
                    if (diff.get(wordSub) == 0) {
                        diff.remove(wordSub);
                    }
                }

                // 如果没有痕迹了，说明 words 的每个 word 都一一对应，不多不少
                if (diff.isEmpty()) {
                    res.add(begin);
                }
            }
        }

        return res;
    }

    public static void main(String[] args) {

        String s = "barfoothefoobarman";
        String[] words = new String[] { "foo", "bar" };

        List<Integer> resoult = findSubstring(s, words);

        System.out.println(resoult);
    }
}
```