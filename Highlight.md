```dataviewjs
// 1. 获取所有页面
let pages = dv.pages('""').filter(p => p.file.path != dv.current().file.path);
let groupedResults = {}; 

// 2. 遍历并提取
for (let page of pages) {
    let content = await app.vault.readRaw(page.file.path);
    // 正则匹配 [!abstract] 块
    let regex = />\s*\[!abstract\]\s*(.*?)(?=\n\n|\n[^>]|$)/gs;
    let match;
    
    while ((match = regex.exec(content)) !== null) {
        let blockText = match[1].replace(/>/g, "").trim();
        
        // 提取块中的所有标签
        let tags = blockText.match(/#[\w\u4e00-\u9fa5]+/g);
        
        // 【关键优化】：创建一个不含标签的纯净版本用于展示
        // 它会把所有的 #标签 替换为空字符串
        let cleanText = blockText.replace(/#[\w\u4e00-\u9fa5]+/g, "").trim();
        
        if (tags) {
            tags.forEach(tag => {
                if (!groupedResults[tag]) groupedResults[tag] = [];
                groupedResults[tag].push([page.file.link, cleanText]);
            });
        } else {
            if (!groupedResults["#未分类"]) groupedResults["#未分类"] = [];
            groupedResults["#未分类"].push([page.file.link, cleanText]);
        }
    }
}

// 3. 按照分组渲染
for (let tag of Object.keys(groupedResults).sort()) {
    dv.header(3, tag); 
    dv.table(["来源笔记", "重点摘要内容"], groupedResults[tag]);
}
```




%% ![[大模型在需求分析与设计中的提效实践#^logic2025]] %%
