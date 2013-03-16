---
layout: post
title: "jquery unique 详解"
date: 2013-03-15 21:22
comments: true
categories: 与JavaScript相关
tags: jquery javascript
keywords: javascript jquery 
description: jquery
---
>最近在工作中使用到了jquery.unique函数,用的过程中出现了很多偏差,所以就查看了jquery源码现在总结一下.    

先看看jQuery.unique(array)api是:   
	删除数组中重复元素.只处理删除DOM元素数组,而不能处理字符串或者数字数组.

再来看看jquery.unique的测试结果:  
![jquery-1.4](/images/common/2013-03-15-jquery-unique/20130315214713.jpg 'jquery-1.4.unique')
[__jquey-1.4__]   
![jquery-1.7](/images/common/2013-03-15-jquery-unique/20130315214630.jpg 'jquery-1.7.2.unique')
[__jquey-1.7.2__]

从测试结果看,1.4版本虽未能够完美实现去除重复元素,但是某些情况(即数组是有序的)下也是能处理数值和字符型数值数组的;而1.7.2版已能完美支持了.

###去除重复元素的实现方法
	$ = function() {
		return {
			unique1: function(elems) {
				for (var i = 0; i < elems.length; ++i) {         // 外层++i
					for (var j = i + 1; j < elems.length; ++j) { // 内层j = i +1
						if (elems[i] === elems[j]) {			 // 判断是否重复的
							elems.splice(i--, 1);				 // 如果重复,就去掉
							break;                               // i-- 与break,保证外层始终从0开始
						}
					}
				}
				return elems;
			},
			unique2: function(elemsWithId) {
				var obj = {};
				for (var i = 0; i < elemsWithId.length; ++i) {
					var elem = elemsWithId[i];
					obj[elem.getAttribute('id')] = obj[elem.getAttribute('id')] || elem;
				}
				elemsWithId.length = 0;
				for (var id in obj) {
					elemsWithId.push(obj[id])
				}
				return elemsWithId;
			}
		}
	}();
方法一使用了两重循环,算法复杂度为`O(n^2)`.实现思路比较直观,即遍历数组,看每个元素是否与后面的元素重复,有重复则移除;但是DOM Element数量较多时性能较差,而jQuery中对大量元素进行去除重复的操作很普遍.   
方法二将Objct当做HashMap/HashSet来使用,算法复杂度为`O(n)`;遗憾的是JavaScript中无法直接用DOM Element作为Object的key,因此只能将id作为key,然而并非所有的 DOM Element 都是有id 的，所以这种方法并不通用。  

<!-- more -->

我们知道,基于比较的排序算法最多可以将算法复杂度降到`O(nlgn)`，（比如结合使用快速排序和插入排序），之后遍历数组时只要比较相邻元素就可以了：    
	unique3: function(sortedElems) {
		for ( var i = 1; i < sortedElems.length; i++ ) {
			if ( sortedElems[i] === sortedElems[ i - 1 ] ) {
				sortedElems.splice( i--, 1 );
			}
		}
		return sortedElems;
	}
JavaScript中有内置的排序算法。因此，在JavaScript中，先排序后去除重复是较好的做法。

###先排序后去除重复
	var sortOrder, siblingCheck;
	sortOrder = function( a, b ) {
		var al, bl,
			ap = [],
			bp = [],
			aup = a.parentNode,
			bup = b.parentNode,
			cur = aup;
		// If the nodes are siblings (or identical) we can do a quick check
		if ( aup === bup ) {
			return siblingCheck( a, b );
		// If no parents were found then the nodes are disconnected
		} else if ( !aup ) {
			return -1;
		} else if ( !bup ) {
			return 1;
		}
		// Otherwise they're somewhere else in the tree so we need
		// to build up a full list of the parentNodes for comparison
		while ( cur ) {
			ap.unshift( cur );
			cur = cur.parentNode;
		}
		cur = bup;
		while ( cur ) {
			bp.unshift( cur );
			cur = cur.parentNode;
		}
		al = ap.length;
		bl = bp.length;
		// Start walking down the tree looking for a discrepancy
		for ( var i = 0; i < al && i < bl; i++ ) {
			if ( ap[i] !== bp[i] ) {
				return siblingCheck( ap[i], bp[i] );
			}
		}
		// We ended someplace up the tree so do a sibling check
		return i === al ?
			siblingCheck( a, bp[i], -1 ) :
			siblingCheck( ap[i], b, 1 );
	};
	siblingCheck = function( a, b, ret ) {
		if ( a === b ) {
			return ret;
		}
		var cur = a.nextSibling;
		while ( cur ) {
			if ( cur === b ) {
				return -1;
			}
			cur = cur.nextSibling;
		}
		return 1;
	};
使用Array内置的sort 方法,并传入了自定义的排序函数sortOrder.   
sortOrder函数的做法是:获取两个被比较元素的所有"直系祖宗",从而确定两个元素在DOM树种的位置;一般来说,两个元素有共同的根,那么就从根元素开始依次向下遍历,直到"分叉点",再对分叉点的元素进行比较.如果直到遍历结束,仍未能到达分叉点(如元素a先遍历,那么i就与al相等),则直接将a与b当前遍历到的"祖宗"进行比较.

###排序时进行检查
优化:先判断元素中是否存在重复的,如果元素不重复,那么就可以不执行后面的遍历并去除重复操作了.实现如下:
	var hasDuplicate = false,
		baseHasDuplicate = true;
	[0, 0].sort(function(){
		baseHasDuplicate = false;
	});
	var uniqueSort = function( results ) {
		if ( sortOrder ) {
			hasDuplicate = baseHasDuplicate;
			results.sort( sortOrder );
			if ( hasDuplicate ) {
				for ( var i = 1; i < results.length; i++ ) {
					if ( results[i] === results[ i - 1 ] ) {
						results.splice( i--, 1 );
					}
				}
			}
		}
		return results;
	};
一个特殊的例外是某些浏览器可能进行了特殊的优化，那么在元素相等时可能就没有调用我们的排序函数了；这种情况下，排序时检查重复的方案就不可行。因此，如果浏览器在元素相等的情况下会调用我们的排序函数，那么就将 hasDuplicate 置为 false，并在排序过程中检查重复；否则，无论如何都视为存在重复，仍然进行遍历去除重复的操作。 

###使用compareDocumentPosition()进行优化
事实上，除 IE 之外浏览器都有内置的 compareDocumentPosition() 方法，用于比较两个 DOM Element 的位置，因此我们可以引入 compareDocumentPosition() 进行优化:   
	if (document.documentElement.compareDocumentPosition) {
		var sort = function(a, b) {
			if (a === b) {
				hasDuplicate = true;
				return 0;
			}
			if (!a.compareDocumentPosition || !b.compareDocumentPosition) {
				return a.compareDocumentPosition ? -1 : 1;
			}
			return a.compareDocumentPosition(b) & 4 ? -1 : 1;
		}
	}
a.compareDocumentPosition(b) & 4 表示取返回值的二进制表示的倒数第三位;   
compareDocumentPosition() 的返回值的每个二进制位表示不同的含义，其中倒数第三位表示元素 a 是否在元素 b 的 “前面”

jQuery 的 unique() 在去除重复前先进行了排序，并使用了多种优化手段，实现了性能较好并且比较通用的去除重复的 DOM Element 的功能。 

###最后的最后
![ff jquery-1.7](/images/common/2013-03-15-jquery-unique/20130315234659.jpg 'ff jquey-1.7')
[__ff jquery-1.7__]
这又是什么原因,你知道了吗?



