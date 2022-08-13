![效果图](https://upload-images.jianshu.io/upload_images/2646856-e2d4d7570092c08a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

spu sku如果不懂的话可以百度，这篇只教你如何实现，整体分为4个步骤，你可以根据自己的业务，灵活使用。

####  1. 获取源数据

数据源的简单模型（这里就不贴实际的数据结构，可以在源码中查看）

````javascript
let metaData=[[1],[2,3,4],[5,6],[7]]
````

#### 2. 生成sku

sku数据的简单模型

```javascript
let sku=[[1,2,5,7],
         [1,2,6,7],
         [1,3,5,7],
         [1,3,6,7],
         [1,4,5,7],
         [1,4,6,7]]
```

可以看的出来最终生成了 1x3x2x1 = 6 条记录

````javascript
descartes(metaData) {
            return metaData.reduce((x, y) => {
                let arr = [];
                x.forEach(x => y.forEach(y => arr.push(x.concat([y]))))
                return arr;
            }, [[]])
        
````

#### 3. 生成表格

生成表格就很简单了，遍历sku生成表格的行和列，并返回通过sku生成的列的下标数组用于表格合并的时候指定列，根据业务需求在** let tdAppend =**替换你自己的内容

````javascript
generatorTable(skuData) {
            let mergeCellColumn=[]
            // 生成表格
            $("thead").html("")
            $("tbody").html("")

            if (skuData[0].length==0){
                return
            }
            skuData.forEach(function (item, index) {
                // 生成表头
                if (index == 0) {
                    let tHeader = "<tr>"
                    item.forEach(function (sub, i) {
                        mergeCellColumn.push(++i);
                        tHeader += "<th>" + sub.specName + "</th>"
                    })
                    tHeader += "<th>价格</th><th>库存</th></tr>"
                    $("thead").append(tHeader)
                }
                // 生成表格内容
                let tr = "<tr>"
                let td = ""
                item.forEach(function (sub) {
                    td += "<td>" + sub.specValueName + "</td>"
                })
                let tdAppend = "<td><input  type='number'  class='form-control'/></td><td><input type='number' class='form-control'/></td>"
                td = td + tdAppend
                tr = tr + td + "</tr>"
                $("tbody").append(tr)
            });
            return mergeCellColumn
        },
````

#### 4. 合并单元格

通过第二步的数据我们可以得到一个6行4列的表格，将每列单独提取出来

````javascript
										 
第一列 [1,1,1,1,1,1]   [1]   
第二列 [2,2,3,3,4,4]   [2,3,4]
第三列 [5,6,5,6,5,6]   [5,6,5,6,5,6]
第四列 [7,7,7,7,7,7]   [7]
````

合并单元其实就是解决数组相邻去重问题，只不过在合并行使用 display:none 而不是真实删除

````javascript
mergeCell(table_id, columns) {
            columns.forEach(function (column) {
                let tds = $(table_id + " tr td:nth-child(" + column + ")"); // 获取当前表格第几列的所有td对象
                let rowSpanIndex=0
                let rowSpanNum = 1;
                for (let i = 0; i < tds.length; i++) {
                    if (i>=tds.length-1){
                        return
                    }
                    if ($(tds[i]).text()==$(tds[i+1]).text()){
                        $(tds[i+1]).hide();
                        $(tds[rowSpanIndex]).attr("rowSpan", ++rowSpanNum);
                    }else{
                        rowSpanIndex=i+1
                        rowSpanNum=1
                    }
                }

            })
        }
````

到这里一个电商sku生成就完成了。
[GirHub源码地址](https://github.com/TobiZh/skuUpload)
