```markdown
# 实验小程序（图片命名 + 词汇范畴判断）使用说明（面向零编程用户）

版本说明
- 该程序为基于浏览器的离线实验实现（jsPsych）。将文件放到同一文件夹后，可在任一现代浏览器中运行。
- 适用于 Windows / macOS / Linux。建议使用 Chrome 或 Edge。

文件与文件夹结构（示例）
- experiment_folder/
  - index.html
  - images_list.csv
  - audio_list.csv
  - images/            ← 放图片文件（26 张）
  - audio/             ← 放语音文件（120 个）
  - README.md

运行前准备（一步步）
1. 将所有 26 张图片放入 experiment_folder/images/。图片尺寸建议 1280×720 px，文件格式常用 .jpg 或 .png。
2. 将所有 120 个语音文件放入 experiment_folder/audio/，常用 .wav 或 .mp3。
3. 打开并编辑 images_list.csv：每行写一个图片文件名（例如：img01.jpg），共 26 行，且文件名与 images/ 中的文件一致。
4. 打开并编辑 audio_list.csv：格式为 CSV（带表头），第一行为：
   filename,language,correct_answer
   然后每行对应一个语音文件，例如：
   word001.wav,德语,1
   word002.wav,汉语,0
   - language 必须写“德语”或“汉语”
   - correct_answer: 对应“有生命”答为 1（或 yes / true），无生命为 0（或 no / false）
5. 启动实验：
   - 最简单（推荐）：在命令行进入 experiment_folder，运行一个小型本地服务器：
     - Python 3: `python -m http.server 8000`
     - 然后在浏览器访问 `http://localhost:8000/index.html`
   - 也可以直接双击 index.html 打开，但部分浏览器对本地文件加载音频/图片有限制，推荐使用本地服务器。
6. 实验流程：
   - 首先填写被试编号与选择被试类型（非熟练 / 较熟练）。
   - 实验一（图片命名）：先练习 6 试次（3 德语 + 3 汉语），再进行 20 个正式试次。
     - 每试次：注视点（500–800 ms 随机）→ 出现图片（最多 3000 ms），图片有外框颜色提示语言（红色 = 汉语；蓝色 = 德语）。被试说出名称并按 J 键，记录反应时；按键或超时后空屏 1000 ms。
   - 实验二（词汇范畴判断）：先练习 12 试次（按你要求的四种条件各 3 个），再进行 108 个正式试次。
     - 每试次：注视点（500–800 ms 随机）→ 播放语音（被试需在 3000 ms 内判断是否为有生命：F = 是，J = 否），记录反应时与正误；按键后空屏 1000 ms。
7. 数据保存：
   - 每个练习/正式阶段结束后，程序会自动生成并提示下载对应 CSV 文件（四个文件）：
     - subj_<timestamp>_exp1_practice.csv
     - subj_<timestamp>_exp1_main.csv
     - subj_<timestamp>_exp2_practice.csv
     - subj_<timestamp>_exp2_main.csv

如何替换图片与语音（不改代码）
- 将图片文件放在 images/ 文件夹；编辑 images_list.csv，将每行替换为对应的文件名（包含扩展名）。程序会自动读取该文件并抽取 6 张练习、20 张正式试验。
- 将语音文件放在 audio/ 文件夹；编辑 audio_list.csv（带 header），确保每行三列：文件名,语言,正确答案。程序会自动分配练习与正式试次并计算 switch/repeat 标签。

数据文件（CSV）中列说明（每列名均以英文小写短语说明）
- subj_id: 被试编号（你在开始界面填写）
- subj_group: 被试类型（non-proficient / proficient）
- experiment: 实验编号（1 = 图片命名，2 = 词汇判断）
- trial_type: 'practice' 或 'main'
- trial_index: 在该部分（practice/main）中的序号（从 1 开始）
- stim_file: 刺激文件名（图片或语音）
- language: 试次语言（'汉语' 或 '德语'）
- seq: 语言序列（'switch' 或 'repeat'；第一试次定义为 'repeat'）
- response_key: 被试按键（'j' 或 'f'），若超时则为空
- response_label: 对词汇任务，为 '是' 或 '否'（图片命名任务无此列或为 null）
- correct_answer: 词汇任务中该试次的正确答案（来自 audio_list.csv），'1'/'0' 或 '是'/'否'
- correct: 布尔，词汇任务中被试是否正确（true/false）
- timeout: 布尔，是否超时（true 表示被试没有在限定时间内按键）
- rt_ms: 反应时（毫秒），若超时则为空
- experiment_end_time / timestamp（CSV 最后一列视保存差异可能存在）：记录保存时间戳

如何直接导入 SPSS 进行三因素方差分析（示例）
- 在 SPSS 中，使用“打开 → 数据”选择 CSV 文件（正式试验文件）。
- 关键因子：
  - 被试间因子（between）：subj_group（将 non-proficient / proficient 设为两水平）
  - 被试内因子（within）：language（汉语/德语）与 seq（switch/repeat）
- DV：rt_ms（反应时）与 error rate（错误率）。在 SPSS 中可先计算各被试每条件的平均反应时以及错误率，然后做 2×2×2 混合方差分析（GLM → 重复测量）。
- CSV 中每一行为一个试次级数据，SPSS 可以先用“数据 → 选择案例 / 变量 → 聚合（Aggregate）”或用“重构 → 将案例转换为变量”得到被试-条件汇总表。

关于 language sequence 的说明（switch/repeat 的定义）
- 程序会根据当前试次语言与上一试次语言是否相同来判断：
  - 如果与上一试次语言不同 -> seq = 'switch'
  - 相同 -> seq = 'repeat'
- 第一试次没有上一试次，我们为了方便统计将其标注为 'repeat'（你也可以在后处理时将其排除）。

练习数据是否进入正式统计
- 练习数据单独保存，程序不会把练习 trial 标记为正式数据；请在 SPSS 分析时只导入 `trial_type == 'main'` 的数据。

若需在线部署（Pavlovia 等）
- 该 HTML/JS 代码可修改为在线部署版本（需做少量适配，如媒体托管），如果你想我可以帮助你把它转为 Pavlovia/其他在线平台版本。

如需我进一步协助（我可以帮你）
- 我可以把这个程序适配成 PsychoPy Builder 项目（图形界面方式）、或把你的实际文件名直接填入 images_list.csv 与 audio_list.csv（如果你把文件名发给我）。
- 若你愿意，把你已经上传的 26 张图片文件名列表和 120 个语音文件名（以及每个语音的正确答案）粘贴给我，我可以替你生成完整填好的 images_list.csv 与 audio_list.csv 并进一步检查语音语言分布是否均衡以便正式实验设计平衡。

祝你毕业论文顺利，若需要我可以：
- 帮你根据实际刺激列表生成已填好的 CSV 文件；
- 或把实验包装为可直接给被试的单文件（例如 Electron 打包），以便无需本地服务器运行。
```