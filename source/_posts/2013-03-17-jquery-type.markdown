---
layout: post
title: "jQuery 数据类型详解"
date: 2013-03-17 15:31
comments: true
categories: 与JavaScript相关
tags: javascript jquery
keywords: 
description: 
---
>ECMAScript中有5种基本数据类型:Undefined,Null,Boolean,Number和String.还有一种复杂数据类型——Object,Object本质上是由一组无序的名值对组成的.

### typeof 操作符
对一个值使用typeof操作符可能返回下列某个字符串:  
"undefined" —— 如果这个值未定义  
"boolean" —— 如果这个值是布尔值  
"string" —— 如果这个值是字符串  
"number" —— 如果这个值是数值  
"object" —— 如果这个值是对象或null  
"function" —— 如果这个值是函数   
	注:typeof 操作符的操作数可以是变量,也可以是数值字面量.typeof是操作符而非函数.
<!--more--> 

### 使用库进行判断数据类型
{% codeblock Javascript Array Syntax lang:js http://j.mp/pPUUmW MDN Documentation %}
	(function($, S, undefined){
		var host = this,
			class2type = {}, // [[Class]] -> type pairs
			OP = Object.prototype,
			toString = OP.toString,
			hasOwn = OP.hasOwnProperty,
			TRUE = true,FALSE = false;
		$ = host && ( host[$] = {} ) ;

		var hasEnumBug = !({toString:1}.propertyIsEnumerable('toString')),
			emumProperties = [
				'hasOwnProperty',
				'isPrototypeOf',
				'propertyIsEnumerable',
				'toString',
				'toLocaleString',
				'valueOf'
			],
			meta = {
				/**
				 * Copies all the properties of s to r
				 * @param r
				 * @param s
				 * @param ov
				 * @param wl
				 * @param deep
				 * @return {*}
				 */
				mix : function(r, s, ov, wl, deep){
					if(!s || !r){return r;}
					if(ov === undefined){ov = true;}
					var i, p, len;
					if(wl && (len = wl.length)){
						for(i = 0; i < len; i++){
							p = wl[i];
							if( p in s){_mix(p, r, s, ov, deep);}
						}
					}else{
						for(p in s){
							_mix(p, r, s, ov, deep);
						}
						if(hasEnumBug){
							for (var j = 0; j < emumProperties.length; j++) {
								p = emumProperties[j];
								if (ov && hasOwn.call(s, p)) {
									r[p] = s[p];
								}
							}
						}
					}
					return r;
				}
			},
			_mix = function(p, r, s, ov, deep){
				if(ov || !(p in r)){
					var target = r[p], src = s[p];
					if(target === src){return ;}
					if( deep && src && (S.isArray(src)) ){
						var clone = target && (S.isArray(target) || S.isPlainObject(target)) ?
							target :
							( S.isArray(src) ? [] : {});
						r[p] = S.mix( clone, src, ov, undefined);
					}else if( src !== undefined ){
						r[p] =s[p];
					}
				}
			},
			seed = ( host && host[S] ) || {};
		S = host[S] = meta.mix( seed, meta );

		/**
		 * If the type of val is null, undefined, number, string, boolean, return true.
		 * @param val
		 * @return {Boolean}
		 */
		function isValidParamValue( val ){
			var t = typeof val;
			return val == null || ( t !== 'object' && t !== 'function' );
		}
		S.mix(KISSY, {
			isNull : function( o ){
				return o === null;
			},
			isUndefined : function( o ){
				return o === undefined;
			},
			isEmptyObject : function(obj){
				for(var name in obj){
					if( name !== undefined ){
						return FALSE;
					}
				}
				return TRUE;
			},
			isPlainObject : function(o){
				return o && toString.call(o) === '[object Object]' && 'isPrototypeOf' in o;
			},
			type : function( o ){
				return o == null ?
					String( o ) : class2type[ toString.call(o) ] || "object";
			},
			each : function(object, fn, context){
				if( object ){
					var key, val, i = 0,
						length = object && object.length,
						isObj = length === undefined || S.type(object) === 'function';
					context = context || null;
					if( isObj ){
						for( key in object ){
							if( fn.call( context, object[key], key, object ) === FALSE ){
								break;
							}
						}
					}else{
						for (val = object[0];
							 i < length && fn.call(context, val, i, object) !== FALSE; val = object[++i]) {
						}
					}
				}
				return object;
			}
		});
		S.each('Boolean Number String Function Array Date RegExp Object'.split(' '),function(name, lc){
			class2type['[object ' + name + ']'] = (lc = name.toLowerCase());
			S['is' + name] = function( o ){
				return S.type(o) == lc;
			}
		});

		$.extend = function( ){
			var options, name, src, copy, copyIsArray, clone,
				target = arguments[0] || {},
				i = 1,
				length = arguments.length,
				deep = false;
			if( typeof target === "boolean" ){
				deep = target;
				target = arguments[1] || {};
				i = 2;
			}
			if( typeof target !== "object" && !$.isFunction(target)){
				target = {};
			}
			if( length === i ){//extend jQuery itself
				target = this;
				--i;
			}
			for( ; i < length; i++){
				if( (options = arguments[i]) != null ){
					for( name in options ){
						src = target[ name ];
						copy = options[ name ];
						if( target === copy ){continue;}
						if( deep && copy && ( $.isPlainObject(copy) || ( copyIsArray = $.isArray(copy) ) ) ){
							if( copyIsArray ){
								copyIsArray = false;
								clone = src && $.isArray(src) ? src : [];
							}else{
								clone = src && $.isPlainObject(src) ? src : {};
							}
							target[ name ] = $.extend( deep, clone, copy );
						}else if( copy !== undefined ){
							target[ name ] = copy;
						}
					}
				}
			}
			return target;
		};
		$.extend({
			isFunction :function(obj){
				return $.type(obj) === 'function';
			},
			isArray : Array.isArray || function(obj){
				return $.type(obj) === 'array';
			},
			isNumeric : function(obj){
				return !isNaN( parseFloat(obj) ) && isFinite(obj);
			},
			isWindow : function(obj){
				return obj != null && obj == obj.window;
			},
			type : function( o ){
				return o == null ?
					String( o ) : class2type[ toString.call(o) ] || "object";
			},
			isEmptyObject : function(obj){
				for(var name in obj){
					if( name !== undefined ){
						return FALSE;
					}
				}
				return TRUE;
			},
			isPlainObject : function(obj){
				if( !obj || $.type(obj) !== "object" || obj.nodeType || $.isWindow(obj) ){
					return false;
				}
				try{
					if( obj.constructor &&
						!hasOwn.call(obj, "constructor") &&
						!hasOwn.call(obj.constructor.prototype, "isPrototypeOf")){
						return false;
					}
				} catch (e){
					return false;
				}
				var key;
				for( key in obj ){}
				return key === undefined || hasOwn.call( obj, key );
			},
			each : function( object, callback, args ){
				var name, i = 0,
					length = object.length,
					isObj = length === undefined || $.isFunction( object );
				if( args ){
					if( isObj ){
						for( name in object ){
							if( callback.apply( object[name] , args) === false ){break;}
						}
					}else{
						for( ; i < length; ){
							if( callback.apply( object[i++], args) === false){break;}
						}
					}
				}else{
					if ( isObj ) {
						for ( name in object ) {
							if ( callback.call( object[ name ], name, object[ name ] ) === false ) {break;}
						}
					} else {
						for ( ; i < length; ) {
							if ( callback.call( object[ i ], i, object[ i++ ] ) === false ) {break;}
						}
					}
				}
				return object;
			}
		});
		$.each("Boolean Number String Function Array Date RegExp Object".split(" "), function(i, name){
			class2type[ "[object "+ name +"]" ] = name.toLowerCase();
		});
	})('jQuery', 'KISSY', undefined);
{% endcodeblock %}
