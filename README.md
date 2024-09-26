<p align="center">
  <a href="https://nightlies.apache.org/flink/flink-cdc-docs-stable/"><img src="docs/static/fig/flinkcdc-logo.png" alt="Flink CDC" style="width: 375px;"></a>
</p>
<p align="center">
<a href="https://github.com/apache/flink-cdc/" target="_blank">
    <img src="https://img.shields.io/github/stars/apache/flink-cdc?style=social&label=Star&maxAge=2592000" alt="Test">
</a>
<a href="https://github.com/apache/flink-cdc/releases" target="_blank">
    <img src="https://img.shields.io/github/v/release/apache/flink-cdc?color=yellow" alt="Release">
</a>
<a href="https://github.com/apache/flink-cdc/actions/workflows/flink_cdc.yml" target="_blank">
    <img src="https://img.shields.io/github/actions/workflow/status/apache/flink-cdc/flink_cdc.yml?branch=master" alt="Build">
</a>
<a href="https://github.com/apache/flink-cdc/tree/master/LICENSE" target="_blank">
    <img src="https://img.shields.io/static/v1?label=license&message=Apache License 2.0&color=white" alt="License">
</a>
</p>


# 基于 3.1.1 版本的功能增强
## 一、支持重启更新内部表 Schema 信息
- 背景：

3.1.1 版本在同步过程中如果业务方对监听的表字段类型做了修改，比如字段是否为空做了更改，当前版本是无法捕捉到的。这样在解析输出到下游后，就会同数据源有差异。

比如：表中有个数值型字段，监听刚开始时不允许为空，开启同步后中途被改成允许为空了，那么CDC会对该字段为NULL的数据基于内部表schema信息覆盖成0。这样业务库对应的列为NULL，而下游是为0。

即使重启也无效。因为 savepoint/checkpoint 保存的表schema信息，只做了新增表时的增加，不支持对表中途schema的修改；且不支持删除无效表，这样理论上state会不断增大。

- 增强功能：

在启动时，重新获取全量所有表的schema信息

