---
title: Interesting solution to Leetcode 804 \"Unique Morse Code Words\"
date: 2022-08-17T21:34:36+08:00
tags: ["leetcode"]
series: ["Leetcode"]
featured: false
---

I created a solution 200\% faster than the "top" 0ms solution on LC using preprocessing and hashing. 
I overengineered this easy problem but I can't find anyone else who used this solution. I thought 
it was interesting so I made a writeup on how it works.

## Question

International Morse Code defines a standard encoding where each letter is mapped to a series of dots and dashes, as follows:

- 'a' maps to ".-",
- 'b' maps to "-...",
- 'c' maps to "-.-.", and so on.

For convenience, the full table for the 26 letters of the English alphabet is given below:

```python
[".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."]
```

Given an array of strings words where each word can be written as a concatenation of the Morse code of each letter, how many unique morse code encodings are there?

For example, "cab" can be written as "-.-..--...", which is the concatenation of "-.-.", ".-", and "-...". We will call such a concatenation the transformation of a word.
Return the number of different transformations among all words we have.

<b>Example 1:</b>
<blockquote>
<b>Input:</b> words = ["gin","zen","gig","msg"]<br>
<b>Output:</b> 2<br>
<b>Explanation:</b> The transformation of each word is:<br>
"gin" -> "--...-."<br>
"zen" -> "--...-."<br>
"gig" -> "--...--."<br>
"msg" -> "--...--."<br>
There are 2 different transformations: "--...-." and "--...--.".<br>
</blockquote>
<b>Example 2:</b>
<blockquote>
<b>Input:</b> words = ["a"]<br>
<b>Output:</b> 1
</blockquote>

<br>
<b>Constraints:</b>
- 1 <= words.length <= 100
- 1 <= words[i].length <= 12
- words[i] consists of lowercase English letters.

## 0ms 100% Efficiency Solution

```java
//gou
public int uniqueMorseRepresentations(String[] words) {

    String[] MORSE = new String[]{
        ".-","-...","-.-.","-..",".","..-.","--.",
        "....","..",".---","-.-",".-..","--","-.",
        "---",".--.","--.-",".-.","...","-","..-",
        "...-",".--","-..-","-.--","--.."};
    
    Set<String> uniqueSet = new HashSet<>();
    for (String word : words) {
        uniqueSet.add(convertToMorseStr(MORSE, word));
    }
    
    return uniqueSet.size();
}

private String convertToMorseStr(String[] MORSE, String word) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < word.length(); i++) {
        sb.append(MORSE[word.charAt(i)-'a']);
    }
    return sb.toString();
}
```

## My Solution

### Part 1 - Precomputing

Hash each of the given morse code strings

```java
static {
    int m = Integer.MAX_VALUE;
    int p = 3;
    int[] P;
    int[] map = new int[26];
    byte[] len = new byte[26];
    
    P = new int[] {p, p * p, p*p*p, p*p*p*p};
    String[] morse = {".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."};
    for (int i = 0; i < 26; i++) {
        int hash = 0;
        int pow = 1;
        for (int ii = 0; ii < morse[i].length(); ii++) {
            hash = (hash + (morse[i].charAt(ii) == '-' ? 2 : 1) * pow) % m;
            pow *= p;
        }
        len[i] = (byte) morse[i].length();
        map[i] = hash;
    }
    System.out.println(Arrays.toString(map));
    System.out.println(Arrays.toString(len));
    System.out.println(Arrays.toString(P));
}
```

### Part 2 - Solution

Take the output of the previous part and hardcode the values in the arrays

```java
public int uniqueMorseRepresentations(String[] words) {
    int m = Integer.MAX_VALUE;
    
    // hashcode of the morse code value of each char
    byte[] map = {7, 41, 50, 14, 1, 49, 17, 40, 4, 79, 23, 43, 8, 5, 26, 52, 71, 16, 13, 2, 22, 67, 25, 68, 77, 44};
    // Length of the morse code value of each char
    byte[] len = {2, 4, 4, 3, 1, 4, 3, 4, 2, 4, 3, 4, 2, 2, 3, 4, 4, 3, 3, 1, 3, 4, 3, 4, 4, 4};
    // P[i] = p ** i
    byte[] P = {1, 3, 9, 27, 81};
    Set<Integer> encodings = new HashSet<>(words.length);
    for (String w : words) {
        int hash = 0;
        for (int i = w.length() - 1; i >= 0; i--) {
            char c = w.charAt(i);
            hash = (hash * P[len[c - 'a']] + map[c - 'a']) % m;
        }
        encodings.add(hash);
    }
    return encodings.size();
}
```

## Explanation

My solution is very similar. It involves hashing each string converted to morse, adding them all to a set, and returning the size of the set.
The difference is that I made serious optimizations to calculating the hash.

The crucial piece of knowledge you need to know is in [this article](https://cp-algorithms.com/string/string-hashing.html) on how to hash a string

<p style="overflow-x: auto">
  \[ 
    hash(s) = s[0] + s[1]*p + s[2]*p^2 + ... + s[n - 1] * p^{n-1} \space\space\space\space mod \space m
  \]
  \[
     = \sum_{i=0}^{n-1}s[i]* p^i \space\space\space\space mod \space m 
  \]
</p>

Typically, s is a string of alphabet letters. "A" is 0, "B" is 1, "C" is 2, "Z" is 26", "a" is "27", "b" is 28, and so on.
In this case, dot is 1 and dash is 2.

Suppose we wanted to encode the string `s = "abc"`.
Encoded in morse code, it is  `s' = ".-" + "-..." + "-.-." = ".--...-.-."`
<p style="overflow-x: auto">
\[
    hash(s') = \textcolor{red}{s'[0] + s'[1]*p} + \textcolor{blue}{s'[2]p^2 + s'[3]*p^3 + s'[4]*p^4 + s'[5]*p^5} + \textcolor{orange}{s'[6]*p^6 + s'[7]*p^7 + s'[8]*p^8 + s'[9]*p^9 + s'[10]*p^{10}}
\]
Each term represents a single char. Group them by which char they belong to (ex. first two belong to "a") and factor them.
\[
    = (\textcolor{red}{s'[0] + s'[1]*p}) + (\textcolor{blue}{s'[2] + s'[3]*p + s'[4]*p^2 + s'[5]*p^3})*p^2 + (\textcolor{orange}{s'[6] + s'[7]*p + s'[8]*p^2 + s'[9]*p^3 + s'[10]*p^4})p^6
\]
Each group is equivalent to a morse code value. s'[0:1] = morse(a) = ".-"
\[
    = (\textcolor{red}{a[0] + a[1]*p}) + (\textcolor{blue}{b[0] + b[1]*p + b[2]*p^2 + b[3]*p^3})*p^2 + (\textcolor{orange}{c[0] + c[1]*p + c[2]*p^2 + c[3]*p^3 + c[4]*p^4})*p^6
\]
Now each group is just hash(morse(s[i]))
\[
    = \textcolor{red}{A} + \textcolor{blue}{B}p^2 + \textcolor{orange}{C}p^6
\]
where A, B, C is the hash of the morse code values of "a", "b", "c"
</p>

This proves that the hash can be computed directly from the original string s as so:

```python
def solution_hash(s: str) -> int:
    p = 3
    i = 0
    hash = 0
    for c in s:
        hash += hash(morse(c)) * p ** i
        i += len(morse(c))
    return hash
```

And this can be used in the solution as so:

```python
def solution(words: List[str]) -> int:
    set = set()
    for s in words:
        set.add(solution_hash(s))
    return len(set)
```

From there, `hash(morse(c))` can be precomputed and `p ** i` can be partially precomputed in a LUT.

### p ** i LUT

If you compute the hash from start to end like in `solution_hash`, the LUT would have to extend to infinity or at the very least do binary exponentiation to compute the exponents. Instead, I did this:

```python
def solution_hash_2(s: str) -> int:
    p = 3
    hash = 0
    for c in reversed(s):
        l = len(morse(c))
        hash = hash(morse(c)) + hash * p ** l
```

This way, `max(l) = max(len(morse(c)))`. In this case, it's only 4, so my LUT table only needs to go up to `p ** 4`.

### Other optimizations

I saved `len(morse(c))` in an array too so I wouldn't have to store the original morse code strings.

## Comparison

The Leetcode test data is not very large so it's very easy to cap out at 0ms. On my own machine, I ran custom test cases 1,000,000 times.

|                    | Mine  | Top Solution |
|--------------------|-------|--------------|
| Default test case  | .096s  | .236s         |
| arr[i].len > 10000 | 40.59s | 96.18s        |


<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>