[apache/poi](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fapache%2Fpoi&objectId=2109604&objectType=1)是apache旗下用于读写Microsoft Office 二进制文件和OOXML 格式文件的开源库。用它来进行excel文件的导出是很趁手的。 一般来说可以直接使用 `Sheet.autoSizeColumn`方法自动调整每列的宽度。但是遇到包含中文的列，`autoSizeColumn`方法计算的列宽是不正确的，算出的宽度不能完整显示中文内容。最近项目中就遇到了这个问题，于是参考网上的各类文章，自己实现了自动适应中文字符宽度的方法

代码如下：

代码语言：javascript

复制

```javascript
	/**
	 * 自动调整列表宽度适应中文字符串
	 * @param sheet
	 * @param startColumnNum 要调整的起始列表号
	 * @param size  要调整的列表数量
	 */
	public static void autoColumnWidthForChineseChar(Sheet sheet, int startColumnNum, int size) {    
	    for (int columnNum = 0; columnNum < size; columnNum++) {
	    	/** 调整每一列宽度 */ 
			sheet.autoSizeColumn(columnNum);
	        /** 获取列宽 */
	        final int columnWidth = sheet.getColumnWidth(columnNum);
	        if(columnNum >= 256*256 ){
	        	/** 列宽已经超过最大列宽则放弃当前列遍历 */
	        	continue;
	        }
	        /** 新的列宽 */
	        int newWidth = columnWidth;
	        /** 遍历所有的行,查找有汉字的列计算新的最大列宽 */
	        for (int rowNum = 0; rowNum <= sheet.getLastRowNum(); rowNum++) {
	            Row currentRow;
	            if (sheet.getRow(rowNum) == null) {
	                continue;
	            } else {
	                currentRow = sheet.getRow(rowNum);
	            }
	            if (currentRow.getCell(columnNum) != null) {
	                Cell currentCell = currentRow.getCell(columnNum);
	                if (currentCell.getCellType() == CellType.STRING) {
	                	String value = currentCell.getStringCellValue();
	                	/** 计算字符串中中文字符的数量 */
	                    int count = chineseCharCountOf(value);
	                    /**在该列字符长度的基础上加上汉字个数计算列宽 */
	                    int length = value.length()*256+count*256*2;
	                	/** 使用字符串的字节长度计算列宽 */
//	                    int length = value.getBytes().length*256;
	                    if (newWidth < length && length < 256*256) {
	                    	newWidth = length;
	                    }
	                }
	            }
	        }
	        if(newWidth != columnWidth){
	        	//设置列宽
	        	sheet.setColumnWidth(columnNum, newWidth);
	        }
	    }
	}
	/**
	 * 计算字符串中中文字符的数量
	 * 参见 <a hrft="https://www.cnblogs.com/straybirds/p/6392306.html">《汉字unicode编码范围》</a>
	 * @param input
	 * @return
	 */
	private static int chineseCharCountOf(String input){
		int count = 0;//汉字数量
		if(null != input){
			String regEx = "[\\u4e00-\\u9fa5]";
			Pattern p = Pattern.compile(regEx);
			Matcher m = p.matcher(input);
			int len = m.groupCount();
			//获取汉字个数
			while (m.find()) {
				for (int i = 0; i <= len; i++) {
					count = count + 1;
				}
			}
		}
		return count;
	}
```

---

**说明**

- 上面的代码中计算汉字数量的方法chineseCharCountOf,为简化实现只统计编译范围在`4e00-u9fa5`的2万多汉字,这也是主要使用的汉字，实际汉字unicode编译的范围并不止这一个，参见 [《汉字unicode编码范围》](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fwww.cnblogs.com%2Fstraybirds%2Fp%2F6392306.html&objectId=2109604&objectType=1),如果更严谨的话，可以把这个方法再完善，将所有的编译范围都包括进去。
- 在网还找到另一个实现就是直接用使用字符串的字节长度计算列宽,不需要统计汉字个数，实际测试效果也是一样的。

代码语言：javascript

复制

```javascript
    	/** 使用字符串的字节长度计算列宽 */
        int length = value.getBytes().length*256;
```

参考资料 [《POI Excel 中文自适用宽度》](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_52628254%2Farticle%2Fdetails%2F120177463&objectId=2109604&objectType=1)