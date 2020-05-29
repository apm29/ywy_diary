---
title: 用分治思想非递归获取N个数组元素的组合
tags:
  - 算法
  - javascript
originContent: ''
categories:
  - 笔记
toc: false
date: 2020-05-29 15:11:16
---

## 问题来源
在计算一些商品规格的时候经常遇到这样的问题：比如一个商品有7个规格键值对 比如 【尺寸：S】 【尺寸 M】【尺寸 L】 【颜色 ：白】【颜色 黑】【套餐：A】【套餐 B】，我们需要计算出所有可能的规格组合，其中规格名称相同的分为一组，可以得到三个数组，用于表示各个规格代表的值：
```javascript
let array = [
	['S', 'M', 'L'],
	['白', '黑'],
        ['A', 'B'],
]
```

分治思想：
> 分治，字面上的解释是“分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，再把子问题分成更小的子问题……直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。在计算机科学中，分治法就是运用分治思想的一种很重要的算法。分治法是很多高效算法的基础，如排序算法（快速排序，归并排序），傅立叶变换（快速傅立叶变换）等等。

## 问题思考

如何把分治的思想用于组合多个数组的元素呢，首先想到的就是把问题先简化，二维数组array只有一个元素的时候是什么样的，只有两个元素的时候是怎么样的，只有两个元素时：用两层循环就能获得全部的组合，那我们继续往下，如果是N个元素的呢，我们可以先把前两个数组的元素组合，形成新的一个数组，如此循环直到新的二维数组只有两个为止,这样问题就简化成了两个元素的数组元素进行排列
> 3元素--> 2元素
```javascript
let array = [
	[
		["S","白"],
		["S","黑"],
		["M","白"],
		["M","黑"], 
		["L","白"],
		["L","黑"]
	],
	["A","B"]
]
```
js的实现：
```javascript
 //array:[...[]] 二维数组
      calculateCombination:function(array){
        if(array.length === 1){
          return array[0].map(e=>[e])
        }
         let tempArray = [...array]
        while (tempArray.length > 1){
          let newArrayOne = []
          for (let i = 0; i < tempArray[0].length; i++) {
            for (let j = 0; j < tempArray[1].length; j++) {
              let itemZero = tempArray[0][i]
              let itemOne = tempArray[1][j]
              let newArrayItem = []
              if( itemZero instanceof Array){
                newArrayItem = [
                  ...itemZero,
                  itemOne
                ]
              } else {
                newArrayItem = [
                  itemZero,
                  itemOne
                ]
              }
              newArrayOne.push(newArrayItem)
            }
          }
          tempArray = [
            newArrayOne,
            ...tempArray.slice(2)
          ]
        }
        return tempArray
      },

```
