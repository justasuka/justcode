---
title: 多个数据源放到一个Excel的不同sheet里面
date: 2021-10-21 16:38:11
tags: ['excel','java']
cover: https://s3.bmp.ovh/imgs/2021/10/52fd491754edacb4.png
---

+ 需求

  多个数据源，放到一个excel里的不同sheet里

+ 工具

  easypoi 3.2.0



用到的依赖

~~~xml
<dependency>
    <groupId>cn.afterturn</groupId>
    <artifactId>easypoi-base</artifactId>
    <version>3.2.0</version>
</dependency>
<dependency>
    <groupId>cn.afterturn</groupId>
    <artifactId>easypoi-annotation</artifactId>
    <version>3.2.0</version>
</dependency>
~~~

#### 生成一个sheet 

> strs里分别是列名，字段名两个为一组。title 表格标题，sheetName sheet名

~~~java
    // 生成一个sheet
    private Map<String ,Object> generateMap(List data, List<String> strs, String title, String sheetName){
        ExportParams exportParams = new ExportParams(title,sheetName);
        List<Map<String, Object>> list = new ArrayList<>();
        data.forEach(e->{
            list.add(beanToMap(e));
        });
        List<ExcelExportEntity> beanList = new ArrayList<>();
        for (int i = 0; i < strs.size(); i+=2) {
            beanList.add(new ExcelExportEntity(strs.get(i), strs.get(i + 1)));
        }
        Map<String,Object> map1 = new HashMap<>();
        map1.put("title",exportParams);
        map1.put("data",list);
        map1.put("entity",beanList);
        return map1;
    }
~~~

#### 生成excel

> 其中传入文件名，sheet列表

~~~java
    private File generateFile(String temp, List<Map<String, Object>> bl){
        File file1 = new File(FilePath + "/" + temp);
        try {
            Workbook workbook = ExcelUtil.exportExcel(bl, ExcelType.HSSF);
            if(!file1.exists()){
                file1.createNewFile();
            }
            FileOutputStream fileOutputStream = new FileOutputStream(file1);
            workbook.write(fileOutputStream);
            fileOutputStream.close();
            workbook.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return file1;
    }
~~~

#### 主要代码

> 填充数据，构造sheet，将构造好的map添加到list

~~~java
   public void fetchFiles1(WordObject wordObject, HttpServletResponse response){
       	
        HashMap<String, File> map = new HashMap<>();
        List<Map<String, Object>> exportParamList = new ArrayList<>();
       
       
        List<Lei> list = Service.fetchData(Lei);
        Map<String,Object> map1 = generateMap(list, "列1", "驼峰字段名", "列2", "字段名2", "title", "sheetName");
        exportParamList.add(map1);
       // 重复以上的个代码

       
        File file1 = generateFile("excel.xls", exportParamList);
        // 这里excel已经生成好了
        // map.put("excel.xls", file1);
        // 这里是另外一个生成doc的方法
        //File file05 = export(wordObject);
        //map.put("doc.doc", file05);
        dlZip(map, response);
    }
~~~

#### 其他代码

~~~java
    // 传入文件Map，下载压缩包
    private void dlZip(HashMap<String ,File> map, HttpServletResponse response){
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ZipOutputStream zos = new ZipOutputStream(bos);
        map.forEach((key,value)->{
            try {
                zos.putNextEntry(new ZipEntry(key));
                zos.write(getFileDataAsBytes(value));
                zos.closeEntry();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        try {
            zos.close();
            response.setHeader("Connection", "close");
            response.setHeader("Content-Type", "application/octet-stream");
            response.setContentType("application/x-msdownload");
            response.addHeader("Content-Disposition", "attachment;fileName=" + "附件.zip");// 设置文件名
            OutputStream os = response.getOutputStream();
            os.write(bos.toByteArray());
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static byte[] getFileDataAsBytes(File File) throws IOException {
        byte[] bytes;
        try(InputStream input=new FileInputStream(File);
            ByteArrayOutputStream output = new ByteArrayOutputStream()) {
            int n;
            while((n=input.read())!=-1) {
                output.write(n);
            };
            bytes=output.toByteArray();
        };
        return bytes;
    }


    private static Workbook getWorkbook(ExcelType type, int size) {
        if (ExcelType.HSSF.equals(type)) {
            return new HSSFWorkbook();
        } else if (size < 100000) {
            return new XSSFWorkbook();
        } else {
            return new SXSSFWorkbook();
        }
    }
    public static Workbook exportExcel(List<Map<String, Object>> list, ExcelType type) {
        Workbook workbook = getWorkbook(type,0);
        for (Map<String, Object> map : list) {
            ExcelExportService service = new ExcelExportService();
            service.createSheetForMap(workbook, (ExportParams) map.get("title"),
                    (List<ExcelExportEntity>) map.get("entity"), (Collection<?>) map.get("data"));
        }
        return workbook;
    }
~~~

>在导出excel的过程中，poi 3.0 和 poi 4.0   有很大的不同，使用时一定要统一好版本。
>
>excel版本新的xlsx对应的XSSF，旧的xls对应的HSSF。 不同的版本转换也比较麻烦。一定要确定好版本。
