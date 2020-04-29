---
layout: post
title: 你一定没用过最简单的使用SXSSFWorkbook快速导出百万条数据
date: 2020-04-29
Author: minweikai
tags: [工具]
comments: true
---

## 常见的导出可能上百万甚至千万的数据量业务场景

1. 历史订单的导出
2. 历史订单明细的导出
3. 历史支付明细的导出
4. 用户信息的导出
5. 等等等

**遇到这些问题我想你一定头疼过，客服或财务可能会在你睡觉的时候找上你**

财务： “大王赶快起来看下系统，系统又卡住了”

大王：“你小子又干啥了”

财务： “我刚导出了一个月的订单明细”

大王：“那应该，没啥问题呀，我们一个月撑死也就1w笔订单”

财务： “上个月不是搞了个活动吗，现在有100w笔订单，然后就导不出来了”

大王默默心里吐槽，但还是从床上爬起来，打开日志一看，果然是这个问题：“内存溢出”

1. 因为往常订单少，根据时间查询订单没有分页，大量数据查询超时
2. 之前使用XSSFWorkbook下载的时候大量数据转换的输出流内存溢出

## 解决方案

1. 对大量数据进行分页查询
2. 导出方法使用SXSSFWorkbook解决

分页就不说了，自己实现

**那如何最简单的使用SXSSFWorkbook快速导出百万条数据呢？**

1. 首先使用造好的轮子：[hutool的ExcelUti](https://hutool.cn/docs/#/poi/Excel%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%94%9F%E6%88%90-BigExcelWriter)

   ```java
   <dependency>
       <groupId>cn.hutool</groupId>
       <artifactId>hutool-all</artifactId>
       <version>5.3.1</version>
   </dependency>
   <dependency>
   	<groupId>org.apache.poi</groupId>
   	<artifactId>poi-ooxml</artifactId>
   	<version>3.17</version>
   </dependency>
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



   **PS. Hutool这个工具很好，错过一定会后悔**

   [简介](https://hutool.cn/docs/#/?id=%E7%AE%80%E4%BB%8B)

   Hutool是一个小而全的Java工具类库，通过静态方法封装，降低相关API的学习成本，提高工作效率，使Java拥有函数式语言般的优雅，让Java语言也可以“甜甜的”。

   Hutool中的工具方法来自于每个用户的精雕细琢，它涵盖了Java开发底层代码中的方方面面，它既是大型项目开发中解决小问题的利器，也是小型项目中的效率担当；

   Hutool是项目中“util”包友好的替代，它节省了开发人员对项目中公用类和公用工具方法的封装时间，使开发专注于业务，同时可以最大限度的避免封装不完善带来的bug。 

2. 测试代码

   ```java
   public static void main(String[] args) {
   
   		//耗时计算
   		TimeInterval timer = DateUtil.timer();
   
   		List<List<Integer>> rows = CollUtil.newArrayList();
   
   		for (int i = 0; i < 1000000; i++) {
   			List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
   			rows.add(list);
   		}
   
   		BigExcelWriter writer = ExcelUtil.getBigWriter("e:/test3.xlsx");
   		// 一次性写出内容，使用默认样式
   		writer.write(rows);
   		// 关闭writer，
   		writer.close();
   
   		System.out.println("执行时间："+timer.intervalSecond());
   	}
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   执行导出100w条数据耗时17s ，注意数据量最大行数：1048576![img](https://img-blog.csdnimg.cn/2020042818313956.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3. 我的结合Hutool在业务中使用

   ```java
   /**
   	 * 导出excel
   	 *
   	 * @param response
   	 * @param name 文件名
   	 * @param rows     业务数据 ArrayList<Map<String, Object>>
   	 */
   	public static void writeExcel(HttpServletResponse response, String name,
   	                              List<Map<String, Object>> rows) {
   		// 通过工具类创建writer
   		ExcelWriter writer = cn.hutool.poi.excel.ExcelUtil.getBigWriter();
   		// 一次性写出内容，使用默认样式
   		writer.write(rows, true);
   		write(response, name, writer);
   	}
   
   /**
   	 * 下载
   	 *
   	 * @param response
   	 * @param name
   	 * @param writer
   	 */
   	public static void write(HttpServletResponse response, String name, ExcelWriter writer) {
   
   		// response为HttpServletResponse对象
   		response.setContentType("application/vnd.ms-excel;charset=utf-8");
   		String fileName = name + ".xls";
   		ServletOutputStream out = null;
   		try {
   			// test.xls是弹出下载对话框的文件名，不能为中文，中文请自行编码
   			response.setHeader("Content-Disposition",
   					"attachment;filename=" + new String(fileName.getBytes("UTF-8"), "ISO8859-1"));
   			out = response.getOutputStream();
   		} catch (IOException e) {
   			e.printStackTrace();
   		}
   		writer.flush(out);
   		// 关闭writer，释放内存
   		writer.close();
   
   	}
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   ```java
   //查出商品数据
   Pager<RespCommoditySalesVo> commoditySalesVos = commoditySalesService.commoditySalesRealTime(suParamsVo);
   		List<Map<String, Object>> rows = CollUtil.newArrayList();
   		commoditySalesVos.getContent().forEach(commoditySales -> {
   			Map<String, Object> row = new LinkedHashMap<>();
               //标题-数据
   			row.put("商品名称", commoditySales.getName());
   			row.put("规格", commoditySales.getWeight());
   			row.put("实时销量", commoditySales.getSales());
   			row.put("剩余可售库存", commoditySales.getInventory());
   			row.put("物料号", commoditySales.getSku());
   			row.put("成本", commoditySales.getCostPrice());
   			row.put("上下架", CommodityShelvesEnum.getValue(commoditySales.getShelves()));
   			rows.add(row);
   		});
   		ExcelUtil.writeExcel(response, "商品实时销量", rows);
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   查询出业务数据组装在map里，key为标题，value为对应数据，然后调用ExcelUtil的write方法写出到excel；

4. 具体业务参数格式可以参照 <https://hutool.cn/docs/#/poi/Excel%E7%94%9F%E6%88%90-ExcelWriter>

## 为什么使用SXSSFWorkbook这么快：

这是它的官方文档，我就不赘述了：<https://poi.apache.org/components/spreadsheet/how-to.html#xssf_sax_api>

懒得点链接看的话，这是谷歌翻译：

SXSSF（软件包：org.apache.poi.xssf.streaming）是XSSF的API兼容流扩展，可用于必须生成非常大的电子表格且堆空间有限的情况。SXSSF通过限制对滑动窗口内的行的访问来实现其低内存占用，而XSSF允许对文档中的所有行进行访问。不再在窗口中的较旧的行将被写入磁盘，因此无法访问。

您可以在工作簿构建时通过*新的SXSSFWorkbook（int windowSize）*指定窗口大小， 也可以通过*SXSSFSheet＃setRandomAccessWindowSize（int windowSize）*在每张纸上设置窗口大小。

当通过createRow（）创建新行并且未刷新记录的总数超过指定的窗口大小时，索引值最低的行将被刷新，并且无法再通过getRow（）进行访问。

默认窗口大小为*100*，由SXSSFWorkbook.DEFAULT_WINDOW_SIZE定义。

windowSize -1表示访问不受限制。在这种情况下，所有未通过调用flushRows（）进行刷新的记录都可以用于随机访问。

请注意，SXSSF分配临时文件，您**必须**始终通过调用dispose方法来明确清理这些文件。

SXSSFWorkbook默认使用内联字符串而不是共享字符串表。这是非常有效的，因为不需要将文档内容保留在内存中，但是众所周知，生成的文档与某些客户端不兼容。启用共享字符串后，文档中的所有唯一字符串都必须保留在内存中。与禁用共享字符串相比，根据您的文档内容，这可能会使用更多的资源。

请注意，根据您正在使用的功能，有些东西仍然可能会占用大量内存，例如，合并区域，超链接，注释等仍然仅存储在内存中，因此，如果广泛使用。

在决定是否启用共享字符串之前，请仔细检查您的内存预算和兼容性需求。