---
icon: arrow-progress
---

# 业务流程

```mermaid
flowchart TD
    upload[文件上传]
    recognize[自动分类+识别]
    verify[人工审核分类和识别结果]
    fetch[调接口获取处理结果]
    
    upload-->docflow
    subgraph docflow[DocFlow自动处理]
    recognize
    end
    docflow-->verify
    verify-->fetch
```