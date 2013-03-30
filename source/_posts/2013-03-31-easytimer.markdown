---
layout: post
title: "通用定时器|倒计时"
date: 2013-03-31 01:14
comments: true
categories: 与JavaScript相关
tags: javascript 
keywords: 与JavaScript相关easytimer
description: 
---
>在一年之前我同事尚谦写了一个倒计时，那个倒计时其实功能上已经很完善了，而且在项目中实际应用效果也不错；今天拿出来整理一下，只为再练练手而已。

#### 调用API
{% codeblock JavaScript Syntax lang:js http://j.mp/pPUUmW MDN Documentation %}
	var _cfg = {'allTimes' : 120, renderTo : 'timer'};
	var timer = new EasyTimer(_cfg);
	timer.start();

	_cfg ={'allTimes' : 40, renderTo : 'timer1', format : '剩余{s}秒'};
		new EasyTimer(_cfg).start();

	_cfg ={'allTimes' : 10, renderTo : 'timer2', format : '剩余{s}秒', ok : function(){alert('我是timer2,我执行完毕了！');}};
		new EasyTimer(_cfg).start();

	_cfg = {'allTimes' : 5, renderTo : 'timer3', immediate: true};
	new EasyTimer(_cfg);

	_cfg = {'allTimes' : 125, renderTo : 'timer4', direction : 1, immediate: true};
	new EasyTimer(_cfg);
{% endcodeblock %}

#### 构造函数及参数说明
	EasyTimer(_cfg) : 构造一个倒计时|定时器
	_cfg = {
		allTimes : 0,  
		defaultFormat :  '{d}天{h}时{m}分{s}秒',
		renderTo : '',
		direction : ''
		immediate : false,
		ok : fn,
		callback : fn
	} 
	allTimes      ：定时器|倒计时的时间总秒数，默认为0
	defaultFormat ：时间显示的格式，不传即显示上述默认格式
	renderTo      ：时间显示的位置元素，必传参数
	direction     ：定时器与倒计时类型区分{0 : 倒计时,1 : 定时器}
	immediate     ：定时器与倒计时初始化后，是否自动执行
	ok            ：定时器与倒计时完成后，调用函数
	callback      ：定时器与倒计时间隔调用函数
	isdebug       ：针对当前定时器与倒计时打印调试信息

<!-- more -->

#### 全部源码
{% codeblock JavaScript Syntax lang:js http://j.mp/pPUUmW MDN Documentation %}
	(function(global){

		function EasyTimer(_cfg){
			var defaultConfig = {
				allTimes : 0,

				day  : 'timer_day',
				hour : 'timer_hour',
				minu : 'timer_minu',
				sec  : 'timer_sec',

				interval : 1,
				defaultFormat :  '{d}天{h}时{m}分{s}秒',
				set : {'s' : 59, 'm' : 59, 'h' : 23},
				direction : 0,
				immediate : false,
				ok : '',
				callback : ''
			};
			this.cfg       = mix(defaultConfig, _cfg);
			this.allTimes = this.cfg.allTimes;

			this.defaultFormat = true;
			if( !this.cfg.format ){
				this.format = this.cfg.defaultFormat;
			}else{
				this.defaultFormat = false;
				this.format = this.cfg.format;
			}
			if( this.cfg.renderTo ){
				this.renderTo = this.cfg.renderTo;
			}else{
				throw new Error("参数中，缺少定时器|倒计时的render元素");
				return ;
			}

			this.day = 0;
			this.hour = 0;
			this.minu = 0;
			this.sec = 0;

			this.interval = this.cfg.interval;
			this.over = false;
			this.direction = this.cfg.direction;

			this.ok = this.cfg.ok;
			this.callback = this.cfg.callback;

			this.invoke = function(){};
			this.init();
		}

		EasyTimer.prototype = {
			init : function(){
				var allTimes = this.allTimes = this.allTimes && parseInt(this.allTimes, 10);
				if( this.cfg.immediate )this.start();
				this.initTimer( this.direction );
				this.showTime('a');
			},
			initTimer : function( direction ){
				var allTimes = this.allTimes
				if( !this.direction ){
					if( allTimes > 0 && allTimes < 60 ){
						this.sec = this.allTimes;
					}else{
						this.day = parseInt(allTimes / 86400, 10);
						allTimes %= 86400;
						this.hour = parseInt(allTimes / 3600, 10);
						allTimes %= 3600;
						this.minu = parseInt(allTimes / 60, 10);
						allTimes %= 60
						this.sec = allTimes;
					}
				}else{
					this.maxDay = parseInt(allTimes / 86400, 10);
				}
				this.invoke = ( !this.direction ) ?
					function (){
						var el = this;
						return arguments[0] - el.interval;
					} :
					function(){
						var el = this;
						return arguments[0] + el.interval;
				};
			},
			start : function(){
				if( this.allTimes > 0){
					this.startTimeCount();
				}else{
					this.over = true;
				}
			},
			startTimeCount : function(){
				var el = this;
				window.setTimeout(function(){
					el.timeCount();
				}, this.interval * 1000);
			},
			timeCount : function(){
				this.updateSec();
				if( !this.over ){
					this.startTimeCount();
				}
			},
			reset : function(direction, e){
				var set = this.cfg.set;
				return ( !direction ) ? set[e] : 0;
			},
			checkNext : function( direction , val, e ){
				var set = this.cfg.set;
				return ( !direction ) ? val > 0 : val < set[e];
			},
			updateSec : function(){
				var next = this.checkNext(this.direction, this.sec, 's');
				if( next ){
					this.sec = this.invoke(this.sec) ;
				}else if( !this.over ){
					this.sec = this.reset(this.direction, 's');
					this.updateMinu();
				}
				this.checkOver();
				this.showTime('s');
			},
			updateMinu : function(){
				var next = this.checkNext(this.direction, this.minu, 'm');
				if( next ){
					this.minu = this.invoke(this.minu);
				}else if( !this.over ){
					this.minu = this.reset(this.direction, 'm');
					this.updateHour();
				}
				this.showTime('m');
			},
			updateHour : function(){
				var next = this.checkNext(this.direction, this.hour, 'h');;
				if( next ){
					this.hour = this.invoke(this.hour);
				}else if( !this.over ){
					this.hour = this.reset(this.direction, 'h');
					this.updateDay();
				}
				this.showTime('h');
			},
			updateDay : function(){
				var day = this.day,
					next = ( !this.direction ) ? day < this.maxDay : day > 0;
				if( next ){
					this.day = this.invoke(this.day);
					this.showTime('d');
				}
			},
			ctinue : function(){
				this.over = false;
				this.startTimeCount();
			},
			stop : function(clear){
				if( clear ){
					this.day = this.hour = this.minu = this.sec = 0;
					this.showTime('a');
					this.pause();
				}
			},
			pause :function(){
				this.over = true;
			},
			checkOver : function(){
				var el = this;
				this.allTimes = this.allTimes - this.interval;
				this.over = this.allTimes <= 0 ? true : false;
				// TODO : 定时器完成方法调用
				if(this.over && isFunction(el.ok)){
					this.ok();
				}
				// TODO : 定时器间隔方法调用
				if( this.callback && isFunction(el.callback) ){
					this.callback(this);
				}
			},
			showTime : function( type ){
				var el = this, split =  ( el.format.split(/\{\w\}/g).length -1 ) >= 1;
				if( this.defaultFormat || split){
					try{
						getElement(el.renderTo).innerHTML = format(el.format, {d : el.day, h : el.hour, m : el.minu, s : el.sec});
						if(this.cfg.debug){
							console.log('格式化结果是:'
								+format(el.format, {d : el.day, h : el.hour, m : el.minu, s : el.sec})
								+"---传入的参数是{ d:"+el.day +', h:'+ el.hour +', m:' + el.minu +', s:'+ el.sec +'}');
						}
					}catch (err){
						alert('更新时间出错，异常类型是'+ err.name +'错误信息是：'+err.message);
					}
				}else{
					var origin = {
						d : [el.cfg.day, el.day],
						h : [el.cfg.hour, el.hour],
						m : [el.cfg.minu, el.minu],
						s : [el.cfg.sec, el.sec]
					};
					if(origin[type]){
						getElement(origin[type][0]).innerHTML = padZero(origin[type][1]);
					}else{
						for(var p in origin){
							if( origin.hasOwnProperty(p)){
								getElement(origin[p][0]).innerHTML = padZero(origin[p][1]);
							}
						}
					}
				}
			}
		};
		// Helpers
		// --------------------------------------------------------------------------
		function mix(r, s, wl){
			for(var p in s){
				if( s.hasOwnProperty(p) ){
					if( wl && indexOf(wl, p) === -1)continue;
					if( p !== 'prototype' ){
						r[p] = s[p];
					}
				}
			}
			return r;
		}
		var toString = Object.prototype.toString;
		var isFunction = function( val ){
			//console.log('val==='+ val.toSource() +',是否是函数：'+(toString.call( val ) === '[object Function]'));
			return toString.call( val ) === '[object Function]';
		};
		var arrProto = Array.prototype;
		var indexOf = arrProto.indexOf ?
				function( arr, item ){
					return arr.indexOf(item);
				} :
				function( arr, item ){
					for(var i = 0, len = arr.length; i< len; i++){
						if( arr[i] === item ){
							return i;
						}
					}
					return -1;
				};
		function format( origin ){
			var args = arrProto.slice.call(arguments, 1);
			var REG_FORMAT_TOKEN = /\{(\w)\}/g;
			return origin.replace(REG_FORMAT_TOKEN, function( m, i ){
				return args[ 0 ][ i ] ? padZero(args[ 0 ][ i ]) : '00';
			});
		}
		function padZero(num, len, pos){
			len = len || 2;
			if(num.toString().length >= len){return num;}
			if( pos ){
			   return  padZero( num + '0', len, pos);
			}
			return padZero( '0' + num , len, pos);
		}
		function getElement( elemStr ){
			var elem = null;
			elem = elemStr.charAt('0') === '#'
				? document.getElementById(elemStr.substring(1)) : elemStr.charAt('') === '.' ?
					document.getElementsByClassName(elemStr.substring(1))[0] :
				document.getElementById(elemStr);
			return elem;
		}

		global.EasyTimer = EasyTimer;
	})(this);
{% endcodeblock %}

