# GPT4V + SoM 实践

### 介绍

探索使用 GPT4V 自动化 UI 测试的可能性：通过 SoM 视觉数据标记来提升 GPT4V 的页面识别。通过这个思路进行一个谷歌搜索内容实践。

### 实践

目标：自动播放一首音乐。

操作流程：

1. 打开谷歌页面。
2. 将可以进行操作的元素进行一个边框标记，并添加上序号。
3. 截图，按照以下 prompt 对 GPT4V 进行一个提问。
4. 根据回答进行页面操作
5. 重复步骤 2-4，直到音乐播放。

prompt

![](./1.png)

```
task: play a random song for me.
type ClickAction = { action: "click", element: number };type TypeAction = { action: "type", element: number, text: string };type ScrollAction = { action: "scroll", direction: "up" | "down" };type RequestInfoFromUser = { action: "request-info", prompt: string };type RememberInfoFromSite = { action: "remember-info", info: string };type Done = { action: "done" };
## response format;
{  briefExplanation: string,;  nextAction: ClickAction | TypeAction | ScrollAction | RequestInfoFromUser | RememberInfoFromSite | Done;}
## response examples;
{  "briefExplanation": "I'll type 'funny cat videos' into the search bar", "nextAction": { "action": "type", "element": 11, "text": "funny cat videos" }}
{  "briefExplanation": "Today's doodle looks interesting, I'll click it", "nextAction": { "action": "click", "element": 9 };};
{  "briefExplanation": "I have to login to create a post", "nextAction": { "action": "request-info", "prompt": "What is your login information?" };}
{;  "briefExplanation": "Today's doodle is about Henrietta Lacks, I'll remember that for our blog post", "nextAction": { "action": "remember-info", "info": "Today's doodle is about Henrietta Lacks" };}
## stored info;[]
## instructions;
# observe the screenshot, and think about the next action;
# output your response in a json markdown code block;
```

结果

```json
{
  "briefExplanation": "I'll type 'play a random song' into the search bar",
  "nextAction": {
    "action": "type",
    "element": 4,
    "text": "play a random song"
  }
}
```



prompt

![](./2.png)

```
task: play a random song for me。
type ClickAction = { action: "click", element: number };type TypeAction = { action: "type", element: number, text: string };type ScrollAction = { action: "scroll", direction: "up" | "down" };type RequestInfoFromUser = { action: "request-info", prompt: string };type RememberInfoFromSite = { action: "remember-info", info: string };type Done = { action: "done" };
## response format
{  briefExplanation: string,;  nextAction: ClickAction | TypeAction | ScrollAction | RequestInfoFromUser | RememberInfoFromSite | Done;};
## response examples;
{  "briefExplanation": "I'll type 'funny cat videos' into the search bar", "nextAction": { "action": "type", "element": 11, "text": "funny cat videos" };}
{  "briefExplanation": "Today's doodle looks interesting, I'll click it", "nextAction": { "action": "click", "element": 9 };};
{  "briefExplanation": "I have to login to create a post", "nextAction": { "action": "request-info", "prompt": "What is your login information?" };}
{;  "briefExplanation": "Today's doodle is about Henrietta Lacks, I'll remember that for our blog post", "nextAction": { "action": "remember-info", "info": "Today's doodle is about Henrietta Lacks" };};
## stored info;[];
## instructions;
# observe the screenshot, and think about the next action;
# output your response in a json markdown code block;
```

结果

```json
{
  "briefExplanation": "I'll click the 'Google Search' button to find a random song",
  "nextAction": {
    "action": "click",
    "element": 10
  }
}
```



prompt

![](./3.png)

```
task: play a random song for me。
type ClickAction = { action: "click", element: number };type TypeAction = { action: "type", element: number, text: string };type ScrollAction = { action: "scroll", direction: "up" | "down" };type RequestInfoFromUser = { action: "request-info", prompt: string };type RememberInfoFromSite = { action: "remember-info", info: string };type Done = { action: "done" };
## response format;
{  briefExplanation: string,;  nextAction: ClickAction | TypeAction | ScrollAction | RequestInfoFromUser | RememberInfoFromSite | Done;};;
## response examples;
{  "briefExplanation": "I'll type 'funny cat videos' into the search bar", "nextAction": { "action": "type", "element": 11, "text": "funny cat videos" };};
{  "briefExplanation": "Today's doodle looks interesting, I'll click it", "nextAction": { "action": "click", "element": 9 };};
{  "briefExplanation": "I have to login to create a post", "nextAction": { "action": "request-info", "prompt": "What is your login information?" };};
{  "briefExplanation": "Today's doodle is about Henrietta Lacks, I'll remember that for our blog post", "nextAction": { "action": "remember-info", "info": "Today's doodle is about Henrietta Lacks" };};
## stored info;[];
## instructions;
# observe the screenshot, and think about the next action;
# output your response in a json markdown code block;
```

结果
```json
{
  "briefExplanation": "I'll click on the video '100 Random Songs You Should Listen To' to play a song",
  "nextAction": {
    "action": "click",
    "element": 0
  }
}
```

