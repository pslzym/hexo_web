---
title: Hbase基础API的使用
tags: |-

  - Hbase
permalink: hbaseji-chu-apide-shi-yong
id: 28
updated: '2016-10-03 17:37:19'
date: 2016-10-03 17:35:43
---

##HBsae API使用
* 基础API
* 扫描器API



###基础API 与 扫描器API
1.创建表

	    Configuration conf = new Configuration();
        conf.set("hbase.zookeeper.quorum", "192.168.0.110:2181");

        Connection conn = null;
        conn = ConnectionFactory.createConnection(conf);

        HBaseAdmin hBaseAdmin = (HBaseAdmin)conn.getAdmin();

        createTable("tableName1", "tst1", "tst2", hBaseAdmin);
        
		public static  void createTable(String tableName, String column1, String column2, HBaseAdmin hBaseAdmin)throws IOException{

        //TableName tableName = new TableName();

        HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));

        tableDescriptor.addFamily(new HColumnDescriptor(column1));

        HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(column2);

        hColumnDescriptor.setVersions(3, 3);

        tableDescriptor.addFamily(hColumnDescriptor);

        hBaseAdmin.createTable(tableDescriptor);

    }
    
2.插入数据
		
	   Configuration conf = new Configuration();
   	conf.set("hbase.zookeeper.quorum", "192.168.0.110:2181");

   	//HBaseAdmin hBaseAdmin = new HBaseAdmin(conf);
   	Connection conn = null;
   	conn = ConnectionFactory.createConnection(conf);
     
		putDatas("tableName1", conn);
		
		public static void putDatas(String tableName, Connection connection) throws  IOException{
        //String [] rows =
        Table table = connection.getTable(TableName.valueOf(tableName));

        Put put = new Put("rowKey1_psl01".getBytes());

        put.addColumn("tst1".getBytes(), "tst1_key".getBytes(), "tst1_value".getBytes());

        put.addColumn("tst2".getBytes(), "tst2_key".getBytes(), "tst2_value".getBytes());

        table.put(put);

    }	


3.获取数据

	   Configuration conf = new Configuration();
   	conf.set("hbase.zookeeper.quorum", "192.168.0.110:2181");

   	//HBaseAdmin hBaseAdmin = new HBaseAdmin(conf);
   	Connection conn = null;
   	conn = ConnectionFactory.createConnection(conf);
   	
   	getData("tableName1", conn);

	public  static void getData(String tableName, Connection connection) throws  IOException{
        Get get = new Get("rowKey1_psl01".getBytes());
        get.addColumn("tst1".getBytes(), "tst1_key".getBytes());

        Table table = connection.getTable(TableName.valueOf(tableName));

        Result result = table.get(get);

        List<Cell> cells = result.listCells();

        for (Cell cell : cells){
            System.out.println("qualifier : " + new String(CellUtil.cloneQualifier(cell)));
            System.out.println("value: " + new String(CellUtil.cloneValue(cell)));
        }
    }
    
    
 4.删除数据
    
    	public  static void delData(String tableName, Connection connection) throws IOException{
        //hBaseAdmin.disableTable();
        Delete delete = new Delete("rowKey1_psl01".getBytes());

        delete.addColumn("tst1".getBytes(), "tst1_key".getBytes());

        Table table = connection.getTable(TableName.valueOf(tableName));
        table.delete(delete);
    }
    
  5.扫描表
  
       public static void hbaseScan(String tableName, Connection connection){
        Scan scan = new Scan();
        scan.setCaching(1000);
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

