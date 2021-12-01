## 项目地址：
[https://github.com/Dark-Athena/dbms_mle_ext_pkg-oracle](https://github.com/Dark-Athena/dbms_mle_ext_pkg-oracle)
## 开发目的：
用于简单化oracle 21c mle功能的使用
## 特点：
- 可以轻松的从其他多种来源导入js模块，
- 可以更轻松的定义输入及输出参数，
- 你只需要了解plsql及js两种语言各自的语法，无需了解mle的特定语法

## 注意事项：
由于可自由定义参数名称及参数个数，因此对于输入或输出参数的任意一个列表，暂只支持NUMBER/VARCHAR2参数的混用或CLOB参数的单独使用


## 案例
### 例1：基本用法
```sql
--example 1
declare
  i_parameter mle_js_ext_pkg.parameter_list;
  o_parameter mle_js_ext_pkg.parameter_list;
  i_js_code   clob := '';
begin
  i_parameter('person').p_value := 'World';
  o_parameter('greeting').p_value := null;
  i_js_code := 'var greeting = "Hello, " + person + "!";';
  mle_js_ext_pkg.exec_js(i_js_code   => i_js_code,
                         i_parameter => i_parameter,
                         o_parameter => o_parameter);
  dbms_output.put_line(o_parameter('greeting').p_value);
end;
```
### 例2：定义参数类型为数字（默认VARCHAR2）
```sql
--example 2 number type 
declare
  i_parameter mle_js_ext_pkg.parameter_list;
  o_parameter mle_js_ext_pkg.parameter_list;
  i_js_code   clob := '';
begin
  i_parameter('a').p_value := '2';
  i_parameter('a').p_type := 'NUMBER';
  i_parameter('b').p_value := '5';
  i_parameter('b').p_type := 'NUMBER';
  o_parameter('result1').p_value := null;
  o_parameter('result1').p_type := 'NUMBER';
  o_parameter('result2').p_value := null;
  o_parameter('result2').p_type := 'NUMBER';

  i_js_code := '
var result1 = a+b;
var result2 = a*b;
';
  mle_js_ext_pkg.exec_js(i_js_code   => i_js_code,
                         i_parameter => i_parameter,
                         o_parameter => o_parameter);
  dbms_output.put_line(o_parameter('result1').p_value);
  dbms_output.put_line(o_parameter('result2').p_value);
end;
/
```
### 例3：从一个动态sql导入js模块，输出一个clob参数
```sql
--example 3 import js module from a dynamic sql and export clob values
declare
  i_parameter mle_js_ext_pkg.parameter_list;
  o_parameter mle_js_ext_pkg.parameter_list_clob;
  i_js_source clob ;
  i_js_code   clob ;
begin
  i_parameter('url').p_value := 'https://www.darkathena.top';
  o_parameter('qrcode_pic') := null;
  i_js_code := q'{const code = qrcode(4, 'L');
code.addData(url);
code.make();
qrcode_pic = code.createDataURL(4);
}';
  mle_js_ext_pkg.import_module(i_src => i_js_source,
                               i_sql => q'{select content from mle_js_contents where name='qrcode'}');

  mle_js_ext_pkg.exec_js(i_js_code   => i_js_code,
                         i_parameter => i_parameter,
                         o_parameter => o_parameter,
                         i_js_source => i_js_source);
  dbms_output.put_line(o_parameter('qrcode_pic'));
end;
/
```
### 例4：从一个clob变量导入js模块
```sql
--example 4 import js module from a clob value
declare
  i_parameter mle_js_ext_pkg.parameter_list;
  o_parameter mle_js_ext_pkg.parameter_list_clob;
  i_js_source clob;
  i_js_code   clob;
  l_clob      clob;
begin
  i_parameter('url').p_value := 'https://www.darkathena.top';
  o_parameter('qrcode_pic') := null;
  i_js_code := q'{const code = qrcode(4, 'L');
code.addData(url);
code.make();
qrcode_pic = code.createDataURL(4);
}';
  dbms_lob.createtemporary(l_clob, true);
  select content into l_clob from mle_js_contents where name = 'qrcode';

  mle_js_ext_pkg.import_module(i_src => i_js_source, i_import => l_clob);

  mle_js_ext_pkg.exec_js(i_js_code   => i_js_code,
                         i_parameter => i_parameter,
                         o_parameter => o_parameter,
                         i_js_source => i_js_source);
  dbms_output.put_line(o_parameter('qrcode_pic'));
end;
```
### 例5：从一个url导入js模块（可能需要配置wallet及acl）
```sql
--example 5 import js module from a url

  mle_js_ext_pkg.import_module(i_src => i_js_source,
                               i_url => 'http://cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4/qrcode.min.js');
```
### 例6：从一个文件导入js模块
```sql
--example 6 import js module from a file

  mle_js_ext_pkg.import_module(i_src      => i_js_source,
                               i_dir      => 'DB_BACKUP_DIR',
                               i_filename => 'qrcode.min.js');
```
### 例7：在一次执行中导入多个js模块
```sql
--example 7 import multiple js module 
declare
  i_js_source clob;
begin
  mle_js_ext_pkg.import_module(i_src      => i_js_source,
                               i_dir      => 'DB_BACKUP_DIR',
                               i_filename => '1.js');

  mle_js_ext_pkg.import_module(i_src      => i_js_source,
                               i_dir      => 'DB_BACKUP_DIR',
                               i_filename => '2.js');

  mle_js_ext_pkg.import_module(i_src => i_js_source,
                               i_url => 'http://cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4/qrcode.min.js');

  mle_js_ext_pkg.import_module(i_src => i_js_source,
                               i_sql => q'{select content from mle_js_contents where name='qrcode'}');
end;
/
```

## 相关文章
[【ORACLE】关于21c版本新增plsql包DBMS_MLE的研究](https://www.darkathena.top/archives/about-dbmsmle)
