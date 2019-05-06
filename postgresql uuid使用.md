# 在 PostgreSQL中使用UUID作为主键

## 安装依赖包
``` bash
sudo apt install postgresql-contrib
```
## 为表添加扩展
``` bash
create extension "uuid-ossp";
```
## 验证
``` bash
testdb=> select uuid_generate_v4();
           uuid_generate_v4           
--------------------------------------
 832191a7-414c-4b30-9c96-bc1b0782c784
(1 row)
```
创建一个使用uuid作为主键的表：
``` sql
create table table_name
(
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4()
);
```