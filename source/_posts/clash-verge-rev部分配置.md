---
title: clash verge rev部分配置
Date: 2024-11-14
Categories:
  - 备忘
Tags:
  - clash
  - 片段
   
---

# 全局配置脚本

```javascript
// Define main function (script entry)

function main(config) {
  // 读取现有的规则
  let oldRules = config.rules;
  
  // 定义新的规则
  let newRules = [
    "IP-CIDR,172.172.0.0/16,DIRECT,no-resolve",
    "IP-CIDR,192.168.0.0/16,DIRECT,no-resolve",
    "IP-CIDR,192.192.0.0/16,DIRECT,no-resolve",
    "IP-CIDR,127.0.0.0/24,DIRECT,no-resolve",
    "DOMAIN-SUFFIX,localhost,DIRECT,no-resolve"
  ];
  
  // 将新的规则添加到现有的规则中
  oldRules = newRules.concat(oldRules);
  
  // 更新配置的规则
  config.rules = oldRules;
  
  // 返回修改后的配置
  return config;
}
```

