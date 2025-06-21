# 济南 YOASOBI 观影会企划书

## TODO

- [ ] 整理主办人群注意事项文档
- [x] 截取视频
- [x] 转码
- [ ] 超分片源
- [ ] 5.1 声道
- [ ] 歌词本
- [ ] 字幕
- [x] 海报
- [ ] 易拉宝
- [ ] 签名旗
- [ ] 票根

## 观影内容

YOASOBI 5th ANNIVERSARY DOME LIVE 2024 “超現実” 11.10/東京 東京ドーム （wowow 电视台放送版本）

> 2025.2.15-16 [【上海】YOASOBI ASIA TOUR 2024-2025 “超现实cho-genjitsu” IN SHANGHAI](https://detail.damai.cn/item.htm?spm=a2oeg.home.card_0.ditem_0.692723e1adQzPr&id=862333825617) 日本国内版本

## 国内审批信息

[上海市营业性演出准予许可决定 沪文旅许[2024]4224号](http://wsbs.wgj.sh.gov.cn/shwgj_ywtb/core/web/welcome/index!getDocumentinfobyId.action?id=0000000090d9a1f80191550e87d22830)

## 盈利说明

本次活动完全非盈利，影院包场及应援物费用由全体参观粉丝（包含 staff）均摊，除此之外不收取任何费用

## 影院选择

### 要求

* 音响好
* 隔音好
* 人均 50 左右
* 最低 4k 激光，可考虑 IMAX
* 站起来不会挡住投影

## 前期准备

### 截取视频

从 `00:01:01` 到 `02:39:27`

```shell
ffmpeg -i input.ts -ss 00:01:01 -to 02:39:27 -c copy output.mp4
```

### 转码

`MPEG-TS` &rarr; `MPEG4-H.264` 

```shell
ffmpeg -i input.mp4 -c:v libx264 -c:a copy output.mp4
```

### 超分

截取为每两分钟一段的切片视频

```shell
ffmpeg -i input.mp4 -c copy -map 0 -segment_time 00:2:00 -f segment -reset_timestamps 1 output_%02d.mp4
```

使用 `Topaz` `Protheus` 超分模型 `Apollo` 插帧模型超分到 4K 并插帧到 60 帧

超分时可能有些意外导致无效的 NAL 单位大小，可以通过下面的指令进行检测

```shell
ffmpeg -i 04.mp4 -f null -
```

合并视频

```shell
ffmpeg -f concat -safe 0 -i input.txt -c copy output.mp4
```



## 应援物

* 应援棒

## 全程录像

## 宣传

### 推文

```
✨【济南限定】YOASOBI五周年东蛋LIVE线下观影会！超现実世界再临✨

🎉有没有抢不到上海场的宝子举手！🙋
💔有没有只去了 day1 意犹未尽的宝子跺脚！
😭有没有看完 day2 走不出来的宝子大哭！
——济南 YOASOBI 观影会统统给你补回来！

💫当 28.58G 母带级画质撞上影院级音效
💥当东蛋万人欢呼融入济南同好尖叫
✨这波线下观影会直接让次元壁裂开！

🔥【划重点】
⏰时间：3-4 月神仙档期（具体群内投票确定）
📍地点：山东济南影厅（保密级选址ing）
🎞片源：WOWOW 官方放送母带
⏳时长：纯享版 2h40min

🌈【沉浸式体验】
✅大屏同步打 call 应援
✅配备完善应援攻略
✅精美制作歌词本

🚨【参与方式】
👉🏻戳我私信获取入群通道
⚠️为保障活动安全仅通过微信群通知
⚠️拒绝任何盈利行为

📣特别提醒：
本活动为同好自发组织，非盈利活动，所有费用由参与者均摊
最终解释权归全体 YOASOBI 推所有💙

#YOASOBI济南线下 #超现実东蛋LIVE
#二次元应援团建 #终于不用隔着屏幕嚎了
↓ ↓ ↓ 想和 ikura 隔空合唱的速来报道！
```