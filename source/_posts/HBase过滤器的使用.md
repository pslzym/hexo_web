---
title: HBase过滤器的使用
tags: |-

  - Hbase
permalink: hbaseguo-lu-qi-de-shi-yong
id: 29
updated: '2016-10-04 10:55:40'
date: 2016-10-04 10:53:27
---

##HBsae 过滤器
* 过滤器API

###过滤器的使用
1. 使用过滤器可以提高操作表的效率,HBase中两种数据读取函数get()和scan()都支持过滤器,支持直接访问和通过指定起止行键来访问,但是缺少细粒度的筛选功能,如基于正则表达式对行键或值进行筛选的功能。
2. 可以使用预定义好的过滤器或者是实现自定义过滤器
3. 过滤器在客户端创建,通过RPC传送到服务器端,在服务器端执行过滤操作,把数据返回给客户端

![](/uploads/2016/10/QQ20161004-0.png)
原理图

不使用过滤器的情况下，scan可以通过设置开始行和结束行的方式进行过滤。（其中设置的条件是左闭右开）
scan 'user_info', {STARTROW=>'0001', ENDROW=>'0003'}
缺少细粒度的筛选功能

####过滤器类型和名称

1. Comparision Filters(比较过滤器) (主要针对行键名和列族名进行过滤)
	RowFilter
	FamilyFilter
	QualifierFilter
	ValueFilter
	DependentColumnFilter
	
2. Dedicated Filters(专用过滤器)  （可以针对值进行过滤）
	SingleColumnValueFilter
	SingleColumnValueExcludeFilter
	PrefixFilter
	PageFilter
	KeyOnlyFilter
	FirstKeyOnlyFilter
	TimestampsFilter
	RandomRowFilter
	
3.Decorating Filters(附加过滤器)
SkipFilter
WhileMatchFilters

####过滤器的使用
RowFilter--->BinaryComparator,RegexStringComparator

	public static void filterTest(String tableName, Connection connection){
	        Scan scan = new Scan();
	        scan.setCaching(1000);
	        ////BinaryComparator, BinaryPrefixComparator, BitComparator, LongComparator, NullComparator, RegexStringComparator, SubstringComparator
	//        RowFilter rowFilter = new RowFilter(CompareFilter.CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("0001")));
	        RowFilter rowFilter = new RowFilter(CompareFilter.CompareOp.EQUAL, new RegexStringComparator("000\\w+"));


        scan.setFilter(rowFilter);

        System.out.println("test-----");

        try {
            Table table = connection.getTable(TableName.valueOf(tableName));
            ResultScanner scanner = table.getScanner(scan);
            for (Result result: scanner){
                System.out.println(result);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
PageFilter
	
	public static void PagefilterTest(String tableName, Connection connection){

        //scan.setCaching(1000);

        PageFilter pageFilter = new PageFilter(3);

        byte [] lastRow = null;

        int pageCount = 0;

        try {
            while (++pageCount > 0){

                System.out.println("pageCount: " + pageCount);
                Scan scan = new Scan();
                scan.setFilter(pageFilter);

                if (lastRow != null){
                    scan.setStartRow(lastRow);
                }

                Table table = connection.getTable(TableName.valueOf(tableName));

                ResultScanner scanner = table.getScanner(scan);

                int count = 0;
                for (Result result: scanner){
                    lastRow = result.getRow();

                    //避免每一页最后一个row重复
                    if(++count >= 3){
                        break;
                    }
                    System.out.println(result);
                }

                ////处理最后一页
                if(count < 3){
                    break;
                }

            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

 


