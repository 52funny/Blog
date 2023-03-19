---
title: '全排序算法'
date: 2023-03-03T22:08:28+08:00
draft: false
---

对于全排序算法，我们只要对每个位置进行模拟，然后看每个位置可以放什么数，将对应的数放进位置中，之后不断的递归回溯，这样就可以找到所有的全排列。

<!--more-->

![](/img/permutation.png)

下面给出 rust 代码：

```rust
fn main() {
    let v = vec![1, 2, 3];
    let p = Permutation::new(&v);
    let permutation_result = p.permutation();
    println!("{:?}", permutation_result);
}

#[allow(dead_code)]
pub struct Permutation<'a> {
    vis: Vec<bool>,
    nums: &'a Vec<i32>,
    path: Vec<i32>,
    result: Vec<Vec<i32>>,
}

impl<'a> Permutation<'a> {
    pub fn new(nums: &'a Vec<i32>) -> Self {
        return Self {
            vis: vec![false; nums.len()],
            path: vec![0; nums.len()],
            result: vec![],
            nums,
        };
    }
    pub fn permutation(mut self) -> Vec<Vec<i32>> {
        Self::dfs(
            &self.nums,
            &mut self.vis,
            &mut self.path,
            &mut self.result,
            0,
        );
        self.result
    }
    fn dfs(
        nums: &Vec<i32>,
        vis: &mut Vec<bool>,
        path: &mut Vec<i32>,
        result: &mut Vec<Vec<i32>>,
        index: usize,
    ) {
        if index == nums.len() {
            result.push(path.clone());
        }
        for i in 0..nums.len() {
            if !vis[i] {
                path[index] = nums[i];
                vis[i] = true;
                Self::dfs(nums, vis, path, result, index + 1);
                vis[i] = false;
            }
        }
    }
}

```
