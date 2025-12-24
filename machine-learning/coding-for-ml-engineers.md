# Coding Interview Cheatsheet (For ML Engineers)
## "I Hate LeetCode" Survival Guide

You don't need to be a competitive programmer. You just need to not blank out on a Medium-level array problem.
Focus on these **3 Patterns** and **3 ML Snippets**.

---

## 🏗️ 3 Patterns to Rule Them All

### 1. The HashMap (Dictionary) 🗝️
**Use for**: "Count frequency", "Find two numbers that sum to target", "Group items".
*   **The Trick**: If you need to search for something you've seen before, store it in a dict `{value: index}`.

```python
# Scenario: Find First Unique Character
def firstUniqChar(s):
    count = {}
    for char in s:
        count[char] = count.get(char, 0) + 1
    
    for i, char in enumerate(s):
        if count[char] == 1:
            return i
    return -1
```

### 2. Two Pointers 👈👉
**Use for**: "Reverse string/array", "Move zeros to end", "Palindrome check", "Sorted arrays".
*   **The Trick**: One pointer at `left=0`, one at `right=len-1`. Move them until they meet or satisfy condition.

```python
# Scenario: Check if Palindrome
def isPalindrome(s):
    l, r = 0, len(s) - 1
    while l < r:
        if s[l] != s[r]: return False
        l += 1; r -= 1
    return True
```

### 3. Sliding Window 🪟
**Use for**: "Longest substring", "Max sum subarray of size K".
*   **The Trick**: Expand `right` to include elements. If condition breaks, shrink `left`.

```python
# Scenario: Max Sum of Subarray of size K
def maxSum(nums, k):
    max_s = curr_s = sum(nums[:k])
    for i in range(k, len(nums)):
        curr_s += nums[i] - nums[i-k] # Add new, remove old
        max_s = max(max_s, curr_s)
    return max_s
```

---

## 🧠 ML-Specific "Coding" (Core Knowledge)
Interviewers often ask candidates to implement a metric or layer from scratch to test deep understanding.

### 1. Intersection over Union (IoU) - Object Detection 
```python
def iou(box1, box2):
    # box = [x1, y1, x2, y2]
    # 1. Determine intersection rectangle
    x1 = max(box1[0], box2[0])
    y1 = max(box1[1], box2[1])
    x2 = min(box1[2], box2[2])
    y2 = min(box1[3], box2[3])
    
    # 2. Calculate area
    inter_area = max(0, x2 - x1) * max(0, y2 - y1)
    box1_area = (box1[2] - box1[0]) * (box1[3] - box1[1])
    box2_area = (box2[2] - box2[0]) * (box2[3] - box2[1])
    
    # 3. IoU = Inter / (Area1 + Area2 - Inter)
    return inter_area / float(box1_area + box2_area - inter_area)
```

### 2. Self-Attention (The "Transformer" Question)
```python
import torch.nn.functional as F

def scaled_dot_product_attention(idx, Q, K, V):
    # Q, K, V shape: [batch, seq_len, dim]
    d_k = Q.size(-1)
    
    # 1. Matmul Q and K^T
    scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(d_k)
    
    # 2. Softmax
    weights = F.softmax(scores, dim=-1)
    
    # 3. Matmul with V
    output = torch.matmul(weights, V)
    return output
```

### 3. Numpy Broadcasting (Image Normalization)
```python
import numpy as np
# Normalize image (H, W, C) using Mean and Std
def normalize(img, mean, std):
    # img: (224, 224, 3)
    # mean: (3,)
    return (img - mean) / std # Broadcasting handles the dimensions
```

---

## ⚡ Last Minute Tips
1.  **Clarify Inputs**: "Are the inputs always valid?", "Is it sorted?", "Does it fit in memory?" (Shows seniority).
2.  **Talk While Typing**: Don't go silent. "I'm using a dictionary here to store frequency..."
3.  **Brute Force is Okay**: If stuck, say: "The naive solution is O(N^2) using nested loops. Let me write that first, then optimize." (Better than writing nothing).
