---
layout: post
title:  "Pholcus爬取链家二手房数据"
categories: go
tags:  go 爬虫
---


* content
{:toc}

Pholcus（幽灵蛛）是一款纯Go语言编写的支持分布式的高并发、重量级爬虫软件，定位于互联网数据采集，为具备一定Go或JS编程基础的人提供一个只需关注规则定制的功能强大的爬虫工具。

要用这个框架很简单，下面通过一个爬虫实例讲解：

<!--excerpt-->

## 研究别人写的源码

这个不用多说，不管是写代码还是生活种的其他事，都要先学习别人的长处，代码就要研究别人的优秀源码。对这个爬虫框架，我们要先学习怎么用它。然后在研究框架的实现。

下面别人是别人写的爬虫实例：
https://github.com/henrylee2cn/pholcus_lib；


## 爬取链家上海二手房
1.在pholcus_lib目录下创建对应的目录。然后包含进去，就ok，直接看源码：


```go

package pholcus_lib

// 基础包
import (
	"fmt"
	"strconv"

	"github.com/henrylee2cn/pholcus/app/downloader/request" //必需
	"github.com/henrylee2cn/pholcus/common/goquery"         //DOM解析
	// "github.com/henrylee2cn/pholcus/logs"               //信息输出
	. "github.com/henrylee2cn/pholcus/app/spider" //必需
)

func init() {
	//添加自身到网易菜单
	Blog.Register()
}

var Blog = &Spider{
	Name:        "上海链家",
	Description: "上海链家 【http://sh.lianjia.com/】",
	// Pausetime:    300,
	// Keyin:        KEYIN,
	// Limit:        LIMIT,
	EnableCookie: false,
	RuleTree: &RuleTree{
		Root: func(ctx *Context) {
			ctx.AddQueue(&request.Request{Url: "http://sh.lianjia.com/ershoufang", Rule: "上海链家二手房主页"})
		},

		Trunk: map[string]*Rule{

			"上海链家二手房主页": {
				ParseFunc: func(ctx *Context) {
					i := 1
					for i <= 100 {
						url := "http://sh.lianjia.com/ershoufang/d" + strconv.Itoa(i)
						ctx.AddQueue(&request.Request{Url: url, Rule: "链接二手房"})
						i++
					}
				},
			},

			"链接二手房": {
				ParseFunc: func(ctx *Context) {
					query := ctx.GetDom()
					query.Find(".list-wrap ul li .info-panel h2 a").Each(func(i int, s *goquery.Selection) {
						if url, ok := s.Attr("href"); ok {
							str := "http://sh.lianjia.com"
							str = str + (url)
							fmt.Println(str)

							ctx.AddQueue(&request.Request{Url: str, Rule: "链接二手房详细"})
						}
					})

				},
			},
			"链接二手房详细": {
				//注意：有无字段语义和是否输出数据必须保持一致
				ItemFields: []string{
					"title",      //标题
					"totalPrice", //总价
					"area",       //面积
					"houseType",  //户型

					"address",      //地址
					"areaEllipsis", //区域

				},
				ParseFunc: func(ctx *Context) {
					query := ctx.GetDom()

					downPayment := ""
					monthPayment := ""
					address := ""
					areaEllipsis := ""
					//获取标题
					title := query.Find(".title h1").Text()
					//获取总价
					totalPrice := query.Find(".price .mainInfo").Text()

					//获取面积
					area := query.Find(".area .mainInfo").Text()

					//户型
					houseType := query.Find(".room .mainInfo").Text()

					//首付

					query.Find(".aroundInfo tr").Each(func(i int, s *goquery.Selection) {

						if i == 4 {
							areaEllipsis = s.Find("td .f1 .areaEllipsis").Text()
						} else if i == 5 {
							address = s.Find("td .addrEllipsis").Text()
						}
					})

					// 结果存入Response中转
					ctx.Output(map[int]interface{}{
						0: title,
						1: totalPrice,
						2: area,
						3: houseType,

						4: address,
						5: areaEllipsis,
					})
				},
			},
		},
	},
}


```

