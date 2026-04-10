![](https://github.com/user-attachments/assets/f705bf54-5dd2-42ba-af2d-40a0a174b0b0)

<h1 align="center">
OmniDocBench
</h1>

<div align="center">
<a href="./README.md">English</a> | 简体中文

[\[📜 arXiv\]](https://arxiv.org/abs/2412.07626) | [[Dataset (🤗Hugging Face)]](https://huggingface.co/datasets/opendatalab/OmniDocBench) | [[Dataset (OpenDataLab)]](https://opendatalab.com/OpenDataLab/OmniDocBench)
</div>

**OmniDocBench**是一个针对真实场景下多样性文档解析评测集，具有以下特点：
- **文档类型多样**：该评测集涉及981个PDF页面，涵盖9种文档类型、4种排版类型和3种语言类型。覆盖面广，包含学术文献、财报、报纸、教材、手写笔记等；
- **标注信息丰富**：包含15个block级别（文本段落、标题、表格等，总量超过20k）和4个Span级别（文本行、行内公式、角标等，总量超过80k）的文档元素的**定位信息**，以及每个元素区域的**识别结果**（文本Text标注，公式LaTeX标注，表格包含LaTeX和HTML两种类型的标注）。OmniDocBench还提供了各个文档组件的**阅读顺序**的标注。除此之外，在页面和block级别还包含多种属性标签，标注了5种**页面属性标签**、3种**文本属性标签**和6种**表格属性标签**。
- **标注质量高**：经过人工筛选，智能标注，人工标注及全量专家质检和大模型质检，数据质量较高。
- **配套评测代码**：当前维护的公开分支聚焦端到端评测，并在页级匹配后输出文本、公式、表格、阅读顺序等细粒度指标。

当前公开运行链路支持的评测维度为：
- 文本
- 行间公式
- 表格
- 阅读顺序

目前支持的metric包括：
- Normalized Edit Distance
- BLEU
- METEOR
- TEDS
- CDM

## 更新

[2025/01/16] 更新Marker、Tesseract OCR、StructEqTable版本；新增Docling、OpenOCR、EasyOCR评测；Table部分的Edit Distance计算改成用norm后的字段；新增评测模型版本信息。

## 当前公开仓库范围

当前维护的公开代码路径是 `src/` 下的端到端评测器，默认模板为 [configs/end2end.yaml](./configs/end2end.yaml)。

历史上的单模块评测入口和顶层重复源码目录已经从运行链路中移除。当前仍会被评测实际使用到的文件和目录见 [OPEN_SOURCE_RUNTIME.md](./OPEN_SOURCE_RUNTIME.md)。
下方保留的历史单模块章节仅用于说明原始项目背景，不代表当前公开分支仍可直接运行。

## 评测集介绍

该评测集涉及981个PDF页面，涵盖9种文档类型、4种排版类型和3种语言类型。OmniDocBench具有丰富的标注，包含15个block级别的标注（文本段落、标题、表格等）和4个Span级别的标注（文本行、行内公式、角标等）。所有文本相关的标注框上都包含文本识别的标注，公式包含LaTeX标注，表格包含LaTeX和HTML两种类型的标注。OmniDocBench还提供了各个文档组件的阅读顺序的标注。除此之外，在页面和block级别还包含多种属性标签，标注了5种页面属性标签、3种文本属性标签和6种表格属性标签。

![](https://github.com/user-attachments/assets/f3e53ba8-bb97-4ca9-b2e7-e2530865aaa9)

<details>
  <summary>评测集的数据格式</summary>

评测集的数据格式为JSON，其结构和各个字段的解释如下：

```json
[{
    "layout_dets": [    // 页面元素列表
        {
            "category_type": "text_block",  // 类别名称
            "poly": [
                136.0, // 位置信息，分别是左上角、右上角、右下角、左下角的x,y坐标
                781.0,
                340.0,
                781.0,
                340.0,
                806.0,
                136.0,
                806.0
            ],
            "ignore": false,        // 是否在评测的时候不考虑
            "order": 0,             // 阅读顺序
            "anno_id": 0,           // 特殊的标注ID，每个layout框唯一
            "text": "xxx",          // 可选字段，OCR结果会写在这里
            "latex": "$xxx$",       // 可选字段，formula和table的LaTeX会写在这里
            "html": "xxx",          // 可选字段，table的HTML会写在这里
            "attribute" {"xxx": "xxx"},         // layout的分类属性，后文会详细展示
            "line_with_spans:": [   // span level的标注框
                {
                    "category_type": "text_span",
                    "poly": [...],
                    "ignore": false,
                    "text": "xxx",   
                    "latex": "$xxx$",
                 },
                 ...
            ],
            "merge_list": [    // 只有包含merge关系的标注框内有这个字段，是否包含merge逻辑取决于是否包含单换行分割小段落，比如列表类型
                {
                    "category_type": "text_block", 
                    "poly": [...],
                    ...   // 跟block级别标注的字段一致
                    "line_with_spans": [...]
                    ...
                 },
                 ...
            ]
        ...
    ],
    "page_info": {         
        "page_no": 0,            // 页码
        "height": 1684,          // 页面的宽
        "width": 1200,           // 页面的高
        "image_path": "xx/xx/",  // 标注的页面文件名称
        "page_attribute": {"xxx": "xxx"}     // 页面的属性标签
    },
    "extra": {
        "relation": [ // 具有相关关系的标注
            {  
                "source_anno_id": 1,
                "target_anno_id": 2, 
                "relation": "parent_son"  // figure/table与其对应的caption/footnote类别的关系标签
            },
            {  
                "source_anno_id": 5,
                "target_anno_id": 6,
                "relation_type": "truncated"  // 段落因为排版原因被截断，会标注一个截断关系标签，后续评测的时候会拼接后再作为一整个段落进行评测
            },
        ]
    }
},
...
]
```

</details>

<details>
  <summary>验证集类别</summary>

验证集类别包括：

```
# Block级别标注框
'title'               # 标题
'text_block'          # 段落级别纯文本
'figure',             # 图片类
'figure_caption',     # 图片说明、标题
'figure_footnote',    # 图片注释
'table',              # 表格主体
'table_caption',      # 表格说明和标题
'table_footnote',     # 表格的注释
'equation_isolated',  # 行间公式
'equation_caption',   # 公式序号
'header'              # 页眉
'footer'              # 页脚  
'page_number'         # 页码
'page_footnote'       # 页面注释
'abandon',            # 其他的舍弃类（比如页面中间的一些无关信息）
'code_txt',           # 代码块
'code_txt_caption',   # 代码块说明
'reference',          # 参考文献类

# Span级别标注框
'text_span'           # span级别的纯文本
'equation_ignore',    # 需要忽略的公式类
'equation_inline',    # 行内公式类
'footnote_mark',      #文章的上下角标
```

</details>

<details>
  <summary>验证集属性标签</summary>

页面分类属性包括：
```
'data_source': #PDF类型分类
    academic_literature  # 学术文献
    PPT2PDF # PPT转PDF
    book # 黑白的图书和教材
    colorful_textbook # 彩色图文教材
    exam_paper # 试卷
    note # 手写笔记
    magazine # 杂志
    research_report # 研报、财报
    newspaper # 报纸

'language':#语种
    en # 英文
    simplified_chinese # 简体中文
    en_ch_mixed # 中英混合

'layout': #页面布局类型
    single_column # 单栏
    double_column # 双栏
    three_column # 三栏
    1andmore_column # 一混多，常见于文献
    other_layout # 其他

'watermark'： # 是否包含水印
    true  
    false

'fuzzy_scan': # 是否模糊扫描
    true  
    false

'colorful_backgroud': # 是否包含彩色背景，需要参与识别的内容的底色包含两个以上
    true  
    false
```

标注框级别属性-表格相关属性:

```
'table_layout': # 表格的方向
    vertical #竖版表格
    horizontal #横版表格

'with_span': # 合并单元格
    False
    True

'line':# 表格的线框
    full_line # 全线框
    less_line # 漏线框
    fewer_line # 三线框 
    wireless_line # 无线框

'language': #表格的语种
    table_en  # 英文表格
    table_simplified_chinese  #中文简体表格
    table_en_ch_mixed  #中英混合表格

'include_equation': # 表格是否包含公式
    False
    True

'include_backgroud': # 表格是否包含底色
    False
    True

'table_vertical' # 表格是否旋转90度或270度
    False
    True
```

标注框级别属性-文本段落相关属性: 
```
'text_language': # 文本的段落内语种
    text_en  # 英文
    text_simplified_chinese #简体中文
    text_en_ch_mixed  #中英混合

'text_background':  #文本的背景色
    white # 默认值，白色背景
    single_colored # 除白色外的单背景色
    multi_colored  # 混合背景色

'text_rotate': # 文本的段落内文字旋转分类
    normal # 默认值，横向文本，没有旋转
    rotate90  # 旋转角度，顺时针旋转90度
    rotate180 # 顺时针旋转180度
    rotate270 # 顺时针旋转270度
    horizontal # 文字是正常的，排版是竖型文本
```

标注框级别属性-公式相关属性: 
```
'formula_type': #公式类型
    print  # 打印体
    handwriting # 手写体
```

</details>


## 评测

OmniDocBench开发了一套基于文档组件拆分和匹配的评测方法，对文本、表格、公式、阅读顺序这四大模块分别提供了对应的指标计算，评测结果除了整体的精度结果以外，还提供了分页面以及分属性的精细化评测结果，精准定位模型文档解析的痛点问题。

![](https://github.com/user-attachments/assets/95c88aaa-75dc-432e-891e-17a7d73e024a)

### 环境配置和运行

评测流水线需要 Python 3.10 以及若干系统依赖（TeX Live、ImageMagick、Ghostscript）以支持 CDM 公式指标。提供两种部署方式：

<details>
<summary><b>方式 A：Conda（推荐开发使用）</b></summary>

> 需要 Ubuntu 22.04 / 20.04，至少 8 GB 磁盘空间和 8 GB 内存，root 权限。

**第 1 步 — 创建环境并安装 Python 依赖**

```bash
conda create -n omnidocbench python=3.10 -y
conda activate omnidocbench
git clone <repo_url> && cd Omnidocbench_v1.6
pip install -e .
python -c "from src.core.pipeline import run_config_file; print('OK')"
```

**第 2 步 — 安装 Ghostscript**

CDM 指标需要 Ghostscript 通过 ImageMagick 完成 PDF 到 PNG 的转换。

```bash
sudo apt-get update && sudo apt-get install -y ghostscript
gs --version   # 预期: 9.55.0 (Ubuntu 22.04)
```

**第 3 步 — 安装 TeX Live 2025**

CDM 指标需要 `pdflatex` 并支持 CJK 中文字体。

```bash
cd ~ && wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
tar -xzf install-tl-unx.tar.gz && cd install-tl-*/
sudo ./install-tl   # 交互式安装，全量约 7 GB

echo 'export PATH=/usr/local/texlive/2025/bin/x86_64-linux:$PATH' >> ~/.bashrc
source ~/.bashrc
pdflatex --version | head -2   # 预期: pdfTeX ... (TeX Live 2025)

# 验证 CJK 支持
kpsewhich CJK.sty && kpsewhich c70gkai.fd
# 如果没有输出: sudo tlmgr install cjk cjkutils arphic gkai
```

**第 4 步 — 安装 ImageMagick 7.x**（从源码编译）

Ubuntu 22.04 apt 默认是 ImageMagick 6.x，CDM 需要 7.x。

```bash
sudo apt-get install -y build-essential pkg-config \
  libjpeg-dev libpng-dev libtiff-dev libwebp-dev \
  libfreetype6-dev libfontconfig1-dev

cd /tmp
wget https://github.com/ImageMagick/ImageMagick/archive/refs/tags/7.1.1-47.tar.gz
tar xzf 7.1.1-47.tar.gz && cd ImageMagick-7.1.1-47
./configure --with-modules --enable-shared --with-gslib \
  --with-gs-font-dir=/usr/share/fonts/type1/gsfonts --prefix=/usr/local
make -j$(nproc) && sudo make install && sudo ldconfig
magick --version | head -2   # 预期: ImageMagick 7.1.1-47

# 允许 PDF 读写
POLICY_FILE=$(find /usr/local/etc/ImageMagick-7 -name policy.xml 2>/dev/null | head -1)
[ -n "$POLICY_FILE" ] && sudo sed -i \
  's|<policy domain="coder" rights="none" pattern="PDF" />|<policy domain="coder" rights="read\|write" pattern="PDF" />|' \
  "$POLICY_FILE"
```

**第 5 步 — 验证并运行**

```bash
python -m pytest tools/test_environment_and_smoke.py::TestEnvironmentVersions -v -s
python pdf_validation.py --config configs/end2end.yaml
```

</details>

<details>
<summary><b>方式 B：Docker（推荐用于复现）</b></summary>

预构建 Docker 镜像打包了经过验证的完整运行时（Python 3.10 conda 环境 + TeX Live 2025 + ImageMagick 7.1.1-47 + Ghostscript 9.55.0）。

**拉取镜像**

```bash
docker pull ghcr.io/zeng-weijun/omnidocbench-eval:repro-ubuntu2204
```

**使用自己的数据运行**

```bash
docker run --rm \
  --entrypoint bash \
  -v /path/to/your_gt.json:/workspace/gt/your_gt.json:ro \
  -v /path/to/your_predictions:/workspace/data_md/predictions:ro \
  -v /path/to/output:/workspace/result \
  ghcr.io/zeng-weijun/omnidocbench-eval:repro-ubuntu2204 \
  -c 'cat > configs/custom.yaml << "EOF"
end2end_eval:
  metrics:
    text_block:
      metric: [Edit_dist]
    display_formula:
      metric: [Edit_dist, CDM]
    table:
      metric: [TEDS, Edit_dist]
    reading_order:
      metric: [Edit_dist]
  dataset:
    dataset_name: end2end_dataset
    ground_truth:
      data_path: ./gt/your_gt.json
    prediction:
      data_path: ./data_md/predictions
    match_method: quick_match
    match_workers: 4
    quick_match_truncated_timeout_sec: 300
    timeout_fallback_max_chunk_span: 10
    timeout_fallback_order_penalty: 0.10
EOF
python pdf_validation.py --config configs/custom.yaml'
```

**验证镜像内运行时**

```bash
docker run --rm --entrypoint bash \
  ghcr.io/zeng-weijun/omnidocbench-eval:repro-ubuntu2204 \
  -lc 'bash script/verify_repro_runtime.sh'
```

**从源码构建**（可选）

```bash
bash script/build_repro_docker_image.sh
```

</details>

#### 验证版本

| 组件 | 版本 |
|------|------|
| Python | 3.10.x |
| TeX Live | 2025 |
| pdflatex | 3.141592653-2.6-1.40.28 |
| ImageMagick | 7.1.1-47 |
| Ghostscript | 9.55.0 |

#### Worker 并发配置

评测流水线有三个并行阶段，建议每个阶段的 worker 数设为机器空闲线程数的 1/3 ~ 1/2，避免死锁或 OOM：

| 阶段 | 配置项 | 说明 |
|------|--------|------|
| 页面匹配 | `match_workers` | 文本对齐 |
| CDM 渲染 | `cdm_workers` | 每个 worker 约 1 GB 内存 |
| TEDS 表格 | `teds_workers` | 表格结构相似度 |

#### 运行评测

所有评测输入通过 [configs/end2end.yaml](./configs/end2end.yaml) 配置。修改 `ground_truth.data_path` 和 `prediction.data_path` 指向你的数据，然后运行：

```bash
python pdf_validation.py --config <config_path>
```

### 端到端评测

端到端评测是对模型在PDF页面内容解析上的精度作出的评测。以模型输出的对整个PDF页面解析结果的Markdown作为Prediction。

<table style="width: 92%; margin: auto; border-collapse: collapse;">
  <thead>
    <tr>
      <th rowspan="2">Method Type</th>
      <th rowspan="2">Methods</th>
      <th colspan="2">Text<sup>Edit</sup>↓</th>
      <th colspan="2">Formula<sup>Edit</sup>↓</th>
      <th colspan="2">Formula<sup>CDM</sup>↑</th>
      <th colspan="2">Table<sup>TEDS</sup>↑</th>
      <th colspan="2">Table<sup>Edit</sup>↓</th>
      <th colspan="2">Read Order<sup>Edit</sup>↓</th>
      <th colspan="2">Overall<sup>Edit</sup>↓</th>
    </tr>
    <tr>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
      <th>EN</th>
      <th>ZH</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4">Pipeline Tools</td>
      <td>MinerU-0.9.3</td>
      <td><strong>0.061</strong></td>
      <td><strong>0.211</strong></td>
      <td><strong>0.278</strong></td>
      <td>0.577</td>
      <td>66.9</td>
      <td>49.5</td>
      <td><strong>78.6</strong></td>
      <td>62.1</td>
      <td><strong>0.180</strong></td>
      <td>0.344</td>
      <td><strong>0.079</strong></td>
      <td>0.288</td>
      <td><strong>0.150</strong></td>
      <td><u>0.355</u></td>
    </tr>
    <tr>
      <td>Marker-1.2.3</td>
      <td><u>0.080</u></td>
      <td>0.315</td>
      <td>0.530</td>
      <td>0.883</td>
      <td>20.1</td>
      <td>16.8</td>
      <td>67.6</td>
      <td>49.2</td>
      <td>0.619</td>
      <td>0.685</td>
      <td>0.114</td>
      <td>0.340</td>
      <td>0.336</td>
      <td>0.556</td>
    </tr>
    <tr>
      <td>Mathpix</td>
      <td>0.101</td>
      <td>0.358</td>
      <td><u>0.306</u></td>
      <td><strong>0.454</strong></td>
      <td>71.4</td>
      <td><strong>72.7</strong></td>
      <td><u>77.0</u></td>
      <td><strong>67.1</strong></td>
      <td>0.243</td>
      <td><strong>0.320</strong></td>
      <td><u>0.105</u></td>
      <td>0.275</td>
      <td><u>0.189</u></td>
      <td><strong>0.352</strong></td>
    </tr>
    <tr>
      <td>Docling</td>
      <td>0.416</td>
      <td>0.987</td>
      <td>0.999</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>61.3</td>
      <td>25.0</td>
      <td>0.627</td>
      <td>0.810</td>
      <td>0.313</td>
      <td>0.837</td>
      <td>0.589</td>
      <td>0.909</td>
    </tr>
    <tr>
      <td rowspan="2">Expert VLMs</td>
      <td>GOT-OCR</td>
      <td>0.191</td>
      <td>0.315</td>
      <td>0.360</td>
      <td><u>0.528</u></td>
      <td><strong>81.8</strong></td>
      <td>51.4</td>
      <td>53.2</td>
      <td>47.2</td>
      <td>0.459</td>
      <td>0.520</td>
      <td>0.143</td>
      <td>0.280</td>
      <td>0.289</td>
      <td>0.411</td>
    </tr>
    <tr>
      <td>Nougat</td>
      <td>0.367</td>
      <td>0.998</td>
      <td>0.488</td>
      <td>0.941</td>
      <td>17.4</td>
      <td>16.9</td>
      <td>39.9</td>
      <td>0</td>
      <td>0.572</td>
      <td>1</td>
      <td>0.384</td>
      <td>0.954</td>
      <td>0.453</td>
      <td>0.973</td>
    </tr>
    <tr>
      <td rowspan="3">General VLMs</td>
      <td>GPT4o</td>
      <td>0.146</td>
      <td>0.409</td>
      <td>0.425</td>
      <td>0.606</td>
      <td><u>76.4</u></td>
      <td>48.2</td>
      <td>72.0</td>
      <td>62.9</td>
      <td><u>0.234</u></td>
      <td><u>0.329</u></td>
      <td>0.128</td>
      <td>0.251</td>
      <td>0.233</td>
      <td>0.399</td>
    </tr>
    <tr>
      <td>Qwen2-VL-72B</td>
      <td>0.253</td>
      <td><u>0.251</u></td>
      <td>0.468</td>
      <td>0.572</td>
      <td>54.9</td>
      <td><u>60.9</u></td>
      <td>59.5</td>
      <td><u>66.4</u></td>
      <td>0.551</td>
      <td>0.518</td>
      <td>0.254</td>
      <td><strong>0.223</strong></td>
      <td>0.381</td>
      <td>0.391</td>
    </tr>
    <tr>
      <td>InternVL2-76B</td>
      <td>0.353</td>
      <td>0.29</td>
      <td>0.543</td>
      <td>0.701</td>
      <td>69.8</td>
      <td>49.6</td>
      <td>63.0</td>
      <td>60.2</td>
      <td>0.547</td>
      <td>0.555</td>
      <td>0.317</td>
      <td><u>0.228</u></td>
      <td>0.440</td>
      <td>0.443</td>
    </tr>
  </tbody>
</table>
<p style="text-align: center; margin-top: -4pt;">
  Comprehensive evaluation of document parsing algorithms on OmniDocBench: performance metrics for text, formula, table, and reading order extraction, with overall scores derived from ground truth comparisons.
</p>


更多分属性评测结果在论文中展示。

#### 端到端评测方法-end2end

端到端评测分为两种方式：
- `end2end`: 该方法是用OmniDocBench的JSON文件作为Ground Truth, config文件请参考：[end2end](./configs/end2end.yaml)
- `md2md`: 该方法是用OmniDocBench的markdown格式作为Ground Truth。具体内容将在下一小节*markdown-to-markdown评测*中详述。

我们推荐使用`end2end`的评测方式，因为该方式可以保留sample的类别和属性信息，从而帮助进行特殊类别ignore的操作，以及分属性的结果输出。

`end2end`的评测可以对四个维度进行评测，我们提供了一个end2end的评测结果示例在[result](./result)中，包括:
- 文本段落
- 行间公式
- 表格
- 阅读顺序

<details>
  <summary>end2end.yaml的字段解释</summary>

`end2end.yaml`的配置如下：

```YAML
end2end_eval:          # 指定task名称，端到端评测通用该task
  metrics:             # 配置需要使用的metric
    text_block:        # 针对文本段落的配置
      metric:
        - Edit_dist    # Normalized Edit Distance
        - BLEU         
        - METEOR
    display_formula:   # 针对行间公式的配置
      metric:
        - Edit_dist
        - CDM          # 仅支持导出CDM评测所需的格式，存储在results中
    table:             # 针对表格的配置
      metric:
        - TEDS
        - Edit_dist
    reading_order:     # 针对阅读顺序的配置
      metric:
        - Edit_dist
  dataset:                                       # 数据集配置
    dataset_name: end2end_dataset                # 数据集名称，无需修改
    ground_truth:
      data_path: ./demo_data/omnidocbench_demo/OmniDocBench_demo.json  # OmniDocBench的路径
    prediction:
      data_path: ./demo_data/end2end            # 模型对PDF页面解析markdown结果的文件夹路径
    match_method: quick_match                    # 匹配方式，可选有: no_split/no_split/quick_match
    filter:                                      # 页面级别的筛选
      language: english                          # 需要评测的页面属性以及对应标签
```

`prediction`下的`data_path`输入的是模型对PDF页面解析结果的文件夹路径，路径中保存的是每个页面对应的markdown，文件名与图片名保持一致，仅将.jpg后缀替换成.md。

除了已支持的metric以外，还支持导出[CDM](https://github.com/opendatalab/UniMERNet/tree/main/cdm)评测所需的格式，只需要在metric中配置CDM字段，即可将输出整理为CDM的输入格式，并存储在[result](./result)中。

在端到端的评测中，config里可以选择配置不同的匹配方式，一共有三种匹配方式：
- `no_split`: 不对text block做拆分和匹配的操作，而是直接合并成一整个markdown进行计算，这种方式下，将不会输出分属性的结果，也不会输出阅读顺序的结果；
- `simple_match`: 不进行任何截断合并操作，仅对文本做双换行的段落分割后，直接与GT进行一对一匹配；
- `quick_match`：在段落分割的基础上，加上截断合并的操作，减少段落分割差异对最终结果的影响，通过*Adjacency Search Match*的方式进行截断合并；

我们推荐使用`quick_match`的方式以达到较好的匹配效果，但如果模型输出的段落分割较准确，也可以使用`simple_match`的方式，评测运行会更加迅速。匹配方法通过`config`中的`dataset`字段下的`match_method`字段进行配置。

使用`filter`字段可以对数据集进行筛选，比如将`dataset`下设置`filter`字段为`language: english`，将会仅评测页面语言为英文的页面。更多页面属性请参考*评测集介绍*部分。如果希望全量评测，请注释掉`filter`相关字段。

</details>


#### 端到端评测方法-md2md

markdown-to-markdown评测以模型输出的对整个PDF页面解析结果的Markdown作为Prediction，用OmniDocBench的markdown格式作为Ground Truth。config文件请参考：[md2md](./configs/md2md.yaml)。我们更加推荐使用上一节的`end2end`的方式使用OmniDocBench进行评测，从而保留丰富的属性标注以及ignore逻辑。但是我们依然提供了`md2md`的评测方法，以便于与现有的评测方式对齐。

`md2md`的评测可以对三个维度进行评测，包括:
- 文本段落
- 行间公式
- 表格
- 阅读顺序

<details>
  <summary>md2md.yaml的字段解释</summary>

`md2md.yaml`的配置如下：

```YAML
end2end_eval:          # 指定task名称，端到端评测通用该task
  metrics:             # 配置需要使用的metric
    text_block:        # 针对文本段落的配置
      metric:
        - Edit_dist    # Normalized Edit Distance
        - BLEU         
        - METEOR
    display_formula:   # 针对行间公式的配置
      metric:
        - Edit_dist
        - CDM          # 仅支持导出CDM评测所需的格式，存储在results中
    table:             # 针对表格的配置
      metric:
        - TEDS
        - Edit_dist
    reading_order:     # 针对阅读顺序的配置
      metric:
        - Edit_dist
  dataset:                                               # 数据集配置
    dataset_name: md2md_dataset                          # 数据集名称，无需修改
    ground_truth:                                        # 针对ground truth的数据集配置
      data_path: ./demo_data/omnidocbench_demo/mds       # OmniDocBench的markdown文件夹路径
      page_info: ./demo_data/omnidocbench_demo/OmniDocBench_demo.json          # OmniDocBench的JSON文件路径，主要是用于获取页面级别的属性
    prediction:                                          # 针对模型预测结果的配置
      data_path: ./demo_data/end2end                     # 模型对PDF页面解析markdown结果的文件夹路径
    match_method: quick_match                            # 匹配方式，可选有: no_split/no_split/quick_match
    filter:                                              # 页面级别的筛选
      language: english                                  # 需要评测的页面属性以及对应标签
```

`prediction`下的`data_path`输入的是模型对PDF页面解析结果的文件夹路径，路径中保存的是每个页面对应的markdown，文件名与图片名保持一致，仅将`.jpg`后缀替换成`.md`。

`ground_truth`下的`data_path`输入的是OmniDocBench的markdown文件夹路径，与模型对PDF页面解析结果的markdown文件名一一对应。`ground_truth`下的`page_info`路径输入的是OmniDocBench的JSON文件路径，主要是用于获取页面级别的属性。如果不需要页面级别分属性的评测结果输出，也可以直接将该字段注释掉。但是，如果没有配置`ground_truth`下的`page_info`字段，就无法使用`filter`相关功能。

除此之外的config中字段的解释请参考*端到端评测-end2end*小节。

</details>

### 公式识别评测

OmniDocBench包含每个PDF页面的公式的bounding box信息以及对应的公式识别标注，因此可以作为公式识别评测的benchmark。公式包括行间公式`equation_isolated`和行内公式`equation_inline`，本repo目前提供的例子是行间公式的评测。

<table style="width: 47%;">
  <thead>
    <tr>
      <th>Models</th>
      <th>CDM</th>
      <th>ExpRate@CDM</th>
      <th>BLEU</th>
      <th>Norm Edit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GOT-OCR</td>
      <td>74.1</td>
      <td>28.0</td>
      <td>55.07</td>
      <td>0.290</td>
    </tr>
    <tr>
      <td>Mathpix</td>
      <td><u>86.6</u></td>
      <td>2.8</td>
      <td><b>66.56</b></td>
      <td>0.322</td>
    </tr>
    <tr>
      <td>Pix2Tex</td>
      <td>73.9</td>
      <td>39.5</td>
      <td>46.00</td>
      <td>0.337</td>
    </tr>
    <tr>
      <td>UniMERNet-B</td>
      <td>85.0</td>
      <td><u>60.2</u></td>
      <td><u>60.84</u></td>
      <td><b>0.238</b></td>
    </tr>
    <tr>
      <td>GPT4o</td>
      <td><b>86.8</b></td>
      <td><b>65.5</b></td>
      <td>45.17</td>
      <td><u>0.282</u></td>
    </tr>
    <tr>
      <td>InternVL2-Llama3-76B</td>
      <td>67.4</td>
      <td>54.5</td>
      <td>47.63</td>
      <td>0.308</td>
    </tr>
    <tr>
      <td>Qwen2-VL-72B</td>
      <td>83.8</td>
      <td>55.4</td>
      <td>53.71</td>
      <td>0.285</td>
    </tr>
  </tbody>
</table>
<p>Component-level formula recognition evaluation on OmniDocBench formula subset.</p>


公式识别评测可以参考[formula_recognition](./configs/formula_recognition.yaml)进行配置。 

<details>
  <summary>formula_recognition.yaml的字段解释</summary>

`formula_recognition.yaml`的配置文件如下：

```YAML
recogition_eval:      # 指定task名称，所有的识别相关的任务通用此task
  metrics:            # 配置需要使用的metric
    - Edit_dist       # Normalized Edit Distance
    - CDM             # 仅支持导出CDM评测所需的格式，存储在results中
  dataset:                                                                   # 数据集配置
    dataset_name: omnidocbench_single_module_dataset                         # 数据集名称，如果按照规定的输入格式则不需要修改
    ground_truth:                                                            # 针对ground truth的数据集配置
      data_path: ./demo_data/recognition/OmniDocBench_demo_formula.json      # 同时包含ground truth和模型prediction结果的JSON文件
      data_key: latex                                                        # 存储Ground Truth的字段名，对于OmniDocBench来说，公式的识别结果存储在latex这个字段中
      category_filter: ['equation_isolated']                                 # 用于评测的类别，在公式识别中，评测的category_name是equation_isolated
    prediction:                                                              # 针对模型预测结果的配置
      data_key: pred                                                         # 存储模型预测结果的字段名，这个是用户自定义的
    category_type: formula                                                   # category_type主要是用于数据预处理策略的选择，可选项有：formula/text
```

`metrics`部分，除了已支持的metric以外，还支持导出[CDM](https://github.com/opendatalab/UniMERNet/tree/main/cdm)评测所需的格式，只需要在metric中配置CDM字段，即可将输出整理为CDM的输入格式，并存储在[result](./result)中。

`dataset`的部分，输入的`ground_truth`的`data_path`中的数据格式与OmniDocBench保持一致，仅对应的公式sample下新增一个自定义字段保存模型的prediction结果。通过`dataset`下的`prediction`字段下的`data_key`对存储了prediction信息的字段进行指定，比如`pred`。关于更多OmniDocBench的文件结构细节请参考`评测集介绍`小节。模型结果的输入格式可以参考[OmniDocBench_demo_formula](./demo_data/recognition/OmniDocBench_demo_formula.json)，其格式为：

```JSON
[{
    "layout_dets": [    // 页面元素列表
        {
            "category_type": "equation_isolated",  // OmniDocBench类别名称
            "poly": [    // OmniDocBench位置信息，分别是左上角、右上角、右下角、左下角的x,y坐标
                136.0, 
                781.0,
                340.0,
                781.0,
                340.0,
                806.0,
                136.0,
                806.0
            ],
            ...   // 其他OmniDocBench字段
            "latex": "$xxx$",  // formula的LaTeX会写在这里
            "pred": "$xxx$",   // !! 模型的prediction结果存储在这里，由用户自定义一个新增字段，存储在与ground truth同级
            
        ...
    ],
    "page_info": {...},        // OmniDocBench页面信息
    "extra": {...}             // OmniDocBench标注间关系信息
},
...
]
```

在此提供一个模型infer的脚本供参考：

```PYTHON
import os
import json
from PIL import Image

def poly2bbox(poly):
    L = poly[0]
    U = poly[1]
    R = poly[2]
    D = poly[5]
    L, R = min(L, R), max(L, R)
    U, D = min(U, D), max(U, D)
    bbox = [L, U, R, D]
    return bbox

question = "<image>\nPlease convert this cropped image directly into latex."

with open('./demo_data/omnidocbench_demo/OmniDocBench_demo.json', 'r') as f:
    samples = json.load(f)
    
for sample in samples:
    img_name = os.path.basename(sample['page_info']['image_path'])
    img_path = os.path.join('./Docparse/images', img_name)
    img = Image.open(img_path)

    if not os.path.exists(img_path):
        print('No exist: ', img_name)
        continue

    for i, anno in enumerate(sample['layout_dets']):
        if anno['category_type'] != 'equation_isolated':   # 筛选出行间公式类别进行评测
            continue

        bbox = poly2bbox(anno['poly'])
        im = img.crop(bbox).convert('RGB')
        response = model.chat(im, question)  # 需要根据模型修改传入图片的方式
        anno['pred'] = response              # 直接在对应的annotation下新增字段存储模型的infer结果

with open('./demo_data/recognition/OmniDocBench_demo_formula.json', 'w', encoding='utf-8') as f:
    json.dump(samples, f, ensure_ascii=False)
```

</details>

### 文字OCR评测

OmniDocBench包含每个PDF页面的所有文字的bounding box信息以及对应的文字识别标注，因此可以作为OCR评测的benchmark。文本的标注包含block_level的标注和span_level的标注，都可以用于评测。本repo目前提供的例子是block_level的评测，即文本段落级别的OCR评测。

<table style="width: 90%; margin: auto; border-collapse: collapse; font-size: small;">
  <thead>
    <tr>
      <th rowspan="2">Model Type</th>
      <th rowspan="2">Model</th>
      <th colspan="3">Language</th>
      <th colspan="3">Text background</th>
      <th colspan="4">Text Rotate</th>
    </tr>
    <tr>
      <th><i>EN</i></th>
      <th><i>ZH</i></th>
      <th><i>Mixed</i></th>
      <th><i>White</i></th>
      <th><i>Single</i></th>
      <th><i>Multi</i></th>
      <th><i>Normal</i></th>
      <th><i>Rotate90</i></th>
      <th><i>Rotate270</i></th>
      <th><i>Horizontal</i></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="7" style="text-align: center;"><strong>Pipeline Tools<br>&<br>Expert Vision<br>Models</strong></td>
      <td>PaddleOCR</td>
      <td>0.071</td>
      <td><strong>0.055</strong></td>
      <td><u>0.118</u></td>
      <td><strong>0.060</strong></td>
      <td><strong>0.038</strong></td>
      <td><u>0.085</u></td>
      <td><strong>0.060</strong></td>
      <td><strong>0.015</strong></td>
      <td><u>0.285</u></td>
      <td><strong>0.021</strong></td>
    </tr>
    <tr>
      <td>OpenOCR</td>
      <td>0.07</td>
      <td><u>0.068</u></td>
      <td><strong>0.106</strong></td>
      <td><u>0.069</u></td>
      <td>0.058</td>
      <td><strong>0.081</strong></td>
      <td><u>0.069</u></td>
      <td><u>0.038</u></td>
      <td>0.891</td>
      <td><u>0.025</u></td>
    </tr>
    <tr>
      <td>Tesseract-OCR</td>
      <td>0.096</td>
      <td>0.551</td>
      <td>0.250</td>
      <td>0.439</td>
      <td>0.328</td>
      <td>0.331</td>
      <td>0.426</td>
      <td>0.117</td>
      <td>0.969</td>
      <td>0.984</td>
    </tr>
    <tr>
      <td>EasyOCR</td>
      <td>0.26</td>
      <td>0.398</td>
      <td>0.445</td>
      <td>0.366</td>
      <td>0.287</td>
      <td>0.388</td>
      <td>0.36</td>
      <td>0.97</td>
      <td>0.997</td>
      <td>0.926</td>
    </tr>
    <tr>
      <td>Surya</td>
      <td>0.057</td>
      <td>0.123</td>
      <td>0.164</td>
      <td>0.093</td>
      <td>0.186</td>
      <td>0.235</td>
      <td>0.104</td>
      <td>0.634</td>
      <td>0.767</td>
      <td>0.255</td>
    </tr>
    <tr>
      <td>Mathpix</td>
      <td><u>0.033</u></td>
      <td>0.240</td>
      <td>0.261</td>
      <td>0.185</td>
      <td>0.121</td>
      <td>0.166</td>
      <td>0.180</td>
      <td><u>0.038</u></td>
      <td><strong>0.185</strong></td>
      <td>0.638</td>
    </tr>
    <tr>
      <td>GOT-OCR</td>
      <td>0.041</td>
      <td>0.112</td>
      <td>0.135</td>
      <td>0.092</td>
      <td><u>0.052</u></td>
      <td>0.155</td>
      <td>0.091</td>
      <td>0.562</td>
      <td>0.966</td>
      <td>0.097</td>
    </tr>
    <tr>
      <td rowspan="3" style="text-align: center;"><strong>Vision Language<br>Models</strong></td>
      <td>Qwen2-VL-72B</td>
      <td>0.072</td>
      <td>0.274</td>
      <td>0.286</td>
      <td>0.234</td>
      <td>0.155</td>
      <td>0.148</td>
      <td>0.223</td>
      <td>0.273</td>
      <td>0.721</td>
      <td>0.067</td>
    </tr>
    <tr>
      <td>InternVL2-76B</td>
      <td>0.074</td>
      <td>0.155</td>
      <td>0.242</td>
      <td>0.113</td>
      <td>0.352</td>
      <td>0.269</td>
      <td>0.132</td>
      <td>0.610</td>
      <td>0.907</td>
      <td>0.595</td>
    </tr>
    <tr>
      <td>GPT4o</td>
      <td><strong>0.020</strong></td>
      <td>0.224</td>
      <td><u>0.125</u></td>
      <td>0.167</td>
      <td>0.140</td>
      <td>0.220</td>
      <td>0.168</td>
      <td>0.115</td>
      <td>0.718</td>
      <td>0.132</td>
    </tr>
  </tbody>
</table>


文字OCR评测可以参考[ocr](./configs/ocr.yaml)进行配置。 

<details>
  <summary>ocr.yaml的字段解释</summary>

`ocr.yaml`的配置文件如下：

```YAML
recogition_eval:      # 指定task名称，所有的识别相关的任务通用此task
  metrics:            # 配置需要使用的metric
    - Edit_dist       # Normalized Edit Distance
    - BLEU
    - METEOR
  dataset:                                                                   # 数据集配置
    dataset_name: omnidocbench_single_module_dataset                         # 数据集名称，如果按照规定的输入格式则不需要修改
    ground_truth:                                                            # 针对ground truth的数据集配置
      data_path: ./demo_data/recognition/OmniDocBench_demo_text_ocr.json     # 同时包含ground truth和模型prediction结果的JSON文件
      data_key: text                                                         # 存储Ground Truth的字段名，对于OmniDocBench来说，文本识别结果存储在text这个字段中，所有block level只要包含text字段的annotations都会参与评测
    prediction:                                                              # 针对模型预测结果的配置
      data_key: pred                                                         # 存储模型预测结果的字段名，这个是用户自定义的
    category_type: text                                                      # category_type主要是用于数据预处理策略的选择，可选项有：formula/text
```

`dataset`的部分，输入的`ground_truth`的`data_path`中的数据格式与OmniDocBench保持一致，仅对应的含有text字段的sample下新增一个自定义字段保存模型的prediction结果。通过`dataset`下的`prediction`字段下的`data_key`对存储了prediction信息的字段进行指定，比如`pred`。数据集的输入格式可以参考[OmniDocBench_demo_text_ocr](./demo_data/recognition/OmniDocBench_demo_text_ocr.json)，各个字段含义可以参考*公式识别评测*部分提供的样例。

在此提供一个模型infer的脚本供参考：

```PYTHON
import os
import json
from PIL import Image

def poly2bbox(poly):
    L = poly[0]
    U = poly[1]
    R = poly[2]
    D = poly[5]
    L, R = min(L, R), max(L, R)
    U, D = min(U, D), max(U, D)
    bbox = [L, U, R, D]
    return bbox

question = "<image>\nPlease convert this cropped image directly into latex."

with open('./demo_data/omnidocbench_demo/OmniDocBench_demo.json', 'r') as f:
    samples = json.load(f)
    
for sample in samples:
    img_name = os.path.basename(sample['page_info']['image_path'])
    img_path = os.path.join('./Docparse/images', img_name)
    img = Image.open(img_path)

    if not os.path.exists(img_path):
        print('No exist: ', img_name)
        continue

    for i, anno in enumerate(sample['layout_dets']):
        if not anno.get('text'):             # 筛选出OmniDocBench中包含text字段的annotations进行模型infer
            continue

        bbox = poly2bbox(anno['poly'])
        im = img.crop(bbox).convert('RGB')
        response = model.chat(im, question)  # 需要根据模型修改传入图片的方式
        anno['pred'] = response              # 直接在对应的annotation下新增字段存储模型的infer结果

with open('./demo_data/recognition/OmniDocBench_demo_text_ocr.json', 'w', encoding='utf-8') as f:
    json.dump(samples, f, ensure_ascii=False)
```

</details>

### 表格识别评测

OmniDocBench包含每个PDF页面的公式的bounding box信息以及对应的表格识别标注，因此可以作为表格识别评测的benchmark。表格识别的标注包含HTML和LaTex两种格式，本repo目前提供的例子是HTML格式的评测。

<table style="width: 100%; margin: auto; border-collapse: collapse; font-size: small;">
  <thead>
    <tr>
      <th rowspan="2">Model Type</th>
      <th rowspan="2">Model</th>
      <th colspan="3">Language</th>
      <th colspan="4">Table Frame Type</th>
      <th colspan="4">Special Situation</th>
      <th rowspan="2">Overall</th>
    </tr>
    <tr>
      <th><i>EN</i></th>
      <th><i>ZH</i></th>
      <th><i>Mixed</i></th>
      <th><i>Full</i></th>
      <th><i>Omission</i></th>
      <th><i>Three</i></th>
      <th><i>Zero</i></th>
      <th><i>Merge Cell</i>(+/-)</th>
      <th><i>Formula</i>(+/-)</th>
      <th><i>Colorful</i>(+/-)</th>
      <th><i>Rotate</i>(+/-)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2" style="text-align: center;"><strong>OCR-based Models</strong></td>
      <td>PaddleOCR</td>
      <td><u>76.8</u></td>
      <td>71.8</td>
      <td>80.1</td>
      <td>67.9</td>
      <td>74.3</td>
      <td><u>81.1</u></td>
      <td>74.5</td>
      <td><u>70.6/75.2</u></td>
      <td><u>71.3/74.1</u></td>
      <td>72.7/74.0</td>
      <td>23.3/74.6</td>
      <td>73.6</td>
    </tr>
    <tr>
      <td>RapidTable</td>
      <td><strong>80.0</strong></td>
      <td><strong>83.2</strong></td>
      <td><strong>91.2</strong></td>
      <td><strong>83.0</strong></td>
      <td><strong>79.7</strong></td>
      <td><strong>83.4</strong></td>
      <td>78.4</td>
      <td><strong>77.1/85.4</strong></td>
      <td><strong>76.7/83.9</strong></td>
      <td><strong>77.6/84.9</strong></td>
      <td><u>25.2/83.7</u></td>
      <td><strong>82.5</strong></td>
    </tr>
    <tr>
      <td rowspan="2" style="text-align: center;"><strong>Expert VLMs</strong></td>
      <td>StructEqTable</td>
      <td>72.8</td>
      <td><u>75.9</u></td>
      <td>83.4</td>
      <td>72.9</td>
      <td><u>76.2</u></td>
      <td>76.9</td>
      <td><strong>88</strong></td>
      <td>64.5/81</td>
      <td>69.2/76.6</td>
      <td><u>72.8/76.4</u></td>
      <td><strong>30.5/76.2</strong></td>
      <td><u>75.8</u></td>
    </tr>
    <tr>
      <td>GOT-OCR</td>
      <td>72.2</td>
      <td>75.5</td>
      <td><u>85.4</u></td>
      <td><u>73.1</u></td>
      <td>72.7</td>
      <td>78.2</td>
      <td>75.7</td>
      <td>65.0/80.2</td>
      <td>64.3/77.3</td>
      <td>70.8/76.9</td>
      <td>8.5/76.3</td>
      <td>74.9</td>
    </tr>
    <tr>
      <td rowspan="2" style="text-align: center;"><strong>General VLMs</strong></td>
      <td>Qwen2-VL-7B</td>
      <td>70.2</td>
      <td>70.7</td>
      <td>82.4</td>
      <td>70.2</td>
      <td>62.8</td>
      <td>74.5</td>
      <td><u>80.3</u></td>
      <td>60.8/76.5</td>
      <td>63.8/72.6</td>
      <td>71.4/70.8</td>
      <td>20.0/72.1</td>
      <td>71.0</td>
    </tr>
    <tr>
      <td>InternVL2-8B</td>
      <td>70.9</td>
      <td>71.5</td>
      <td>77.4</td>
      <td>69.5</td>
      <td>69.2</td>
      <td>74.8</td>
      <td>75.8</td>
      <td>58.7/78.4</td>
      <td>62.4/73.6</td>
      <td>68.2/73.1</td>
      <td>20.4/72.6</td>
      <td>71.5</td>
    </tr>
  </tbody>
</table>
<p>Component-level Table Recognition evaluation on OmniDocBench table subset. <i>(+/-)</i> means <i>with/without</i> special situation.</p>


表格识别评测可以参考[table_recognition](./configs/table_recognition.yaml)进行配置。 

**对于模型预测为LaTex格式的表格, 会使用[latexml](https://math.nist.gov/~BMiller/LaTeXML/)工具将latex转为html 再进行评测. 评测代码会自动进行格式转换,需要用户预先安装[latexml](https://math.nist.gov/~BMiller/LaTeXML/)**

<details>
  <summary>table_recognition.yaml的字段解释</summary>

`table_recognition.yaml`的配置文件如下：

```YAML
recogition_eval:      # 指定task名称，所有的识别相关的任务通用此task
  metrics:            # 配置需要使用的metric
    - TEDS            # Tree Edit Distance based Similarity
    - Edit_dist       # Normalized Edit Distance
  dataset:                                                                   # 数据集配置
    dataset_name: omnidocbench_single_module_dataset                         # 数据集名称，如果按照规定的输入格式则不需要修改
    ground_truth:                                                            # 针对ground truth的数据集配置
      data_path: ./demo_data/recognition/OmniDocBench_demo_table.json        # 同时包含ground truth和模型prediction结果的JSON文件
      data_key: html                                                         # 存储Ground Truth的字段名，对于OmniDocBench来说，表格的识别结果存储在html和latex两个字段中, 评测latex格式表格时改为latex
      category_filter: table                                                 # 用于评测的类别，在表格识别中，评测的category_name是table
    prediction:                                                              # 针对模型预测结果的配置
      data_key: pred                                                         # 存储模型预测结果的字段名，这个是用户自定义的
    category_type: table                                                     # category_type主要是用于数据预处理策略的选择
```

`dataset`的部分，输入的`ground_truth`的`data_path`中的数据格式与OmniDocBench保持一致，仅对应的表格sample下新增一个自定义字段保存模型的prediction结果。通过`dataset`下的`prediction`字段下的`data_key`对存储了prediction信息的字段进行指定，比如`pred`。关于更多OmniDocBench的文件结构细节请参考`评测集介绍`小节。模型结果的输入格式可以参考[OmniDocBench_demo_table](./demo_data/recognition/OmniDocBench_demo_table.json)，其格式为：

```JSON
[{
    "layout_dets": [    // 页面元素列表
        {
            "category_type": "table",  // OmniDocBench类别名称
            "poly": [    // OmniDocBench位置信息，分别是左上角、右上角、右下角、左下角的x,y坐标
                136.0, 
                781.0,
                340.0,
                781.0,
                340.0,
                806.0,
                136.0,
                806.0
            ],
            ...   // 其他OmniDocBench字段
            "latex": "$xxx$",  // table的LaTeX标注会写在这里
            "html": "$xxx$",  // table的HTML标注会写在这里
            "pred": "$xxx$",   // !! 模型的prediction结果存储在这里，由用户自定义一个新增字段，存储在与ground truth同级
            
        ...
    ],
    "page_info": {...},        // OmniDocBench页面信息
    "extra": {...}             // OmniDocBench标注间关系信息
},
...
]
```

在此提供一个模型infer的脚本供参考：

```PYTHON
import os
import json
from PIL import Image

def poly2bbox(poly):
    L = poly[0]
    U = poly[1]
    R = poly[2]
    D = poly[5]
    L, R = min(L, R), max(L, R)
    U, D = min(U, D), max(U, D)
    bbox = [L, U, R, D]
    return bbox

question = "<image>\nPlease convert this cropped image directly into html format of table."

with open('./demo_data/omnidocbench_demo/OmniDocBench_demo.json', 'r') as f:
    samples = json.load(f)
    
for sample in samples:
    img_name = os.path.basename(sample['page_info']['image_path'])
    img_path = os.path.join('./demo_data/omnidocbench_demo/images', img_name)
    img = Image.open(img_path)

    if not os.path.exists(img_path):
        print('No exist: ', img_name)
        continue

    for i, anno in enumerate(sample['layout_dets']):
        if anno['category_type'] != 'table':   # 筛选出表格类别进行评测
            continue

        bbox = poly2bbox(anno['poly'])
        im = img.crop(bbox).convert('RGB')
        response = model.chat(im, question)  # 需要根据模型修改传入图片的方式
        anno['pred'] = response              # 直接在对应的annotation下新增字段存储模型的infer结果

with open('./demo_data/recognition/OmniDocBench_demo_table.json', 'w', encoding='utf-8') as f:
    json.dump(samples, f, ensure_ascii=False)
```

</details>


### Layout检测

OmniDocBench包含每个PDF页面的所有文档组件的bounding box信息，因此可以作为Layout检测任务评测的benchmark。

<table style="width: 90%; margin: auto; border-collapse: collapse;">
  <caption>Component-level layout detection evaluation on OmniDocBench layout subset: mAP results by PDF page type.</caption>
  <thead>
    <tr>
      <th style="border-bottom: 2px solid black;">Model</th>
      <th style="border-bottom: 2px solid black;">Book</th>
      <th style="border-bottom: 2px solid black;">Slides</th>
      <th style="border-bottom: 2px solid black;">Research Report</th>
      <th style="border-bottom: 2px solid black;">Textbook</th>
      <th style="border-bottom: 2px solid black;">Exam Paper</th>
      <th style="border-bottom: 2px solid black;">Magazine</th>
      <th style="border-bottom: 2px solid black;">Academic Literature</th>
      <th style="border-bottom: 2px solid black;">Notes</th>
      <th style="border-bottom: 2px solid black;">Newspaper</th>
      <th style="border-bottom: 2px solid black;">Average mAP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border-top: 1px solid black;">DiT-L</td>
      <td style="border-top: 1px solid black; text-decoration: underline;">43.44</td>
      <td style="border-top: 1px solid black; text-decoration: underline;">13.72</td>
      <td style="border-top: 1px solid black; text-decoration: underline;">45.85</td>
      <td style="border-top: 1px solid black;">15.45</td>
      <td style="border-top: 1px solid black;">3.40</td>
      <td style="border-top: 1px solid black; text-decoration: underline;">29.23</td>
      <td style="border-top: 1px solid black; text-decoration: underline;">66.13</td>
      <td style="border-top: 1px solid black;">0.21</td>
      <td style="border-top: 1px solid black;">23.65</td>
      <td style="border-top: 1px solid black;">26.90</td>
    </tr>
    <tr>
      <td>LayoutLMv3</td>
      <td>42.12</td>
      <td>13.63</td>
      <td>43.22</td>
      <td style="text-decoration: underline;">21.00</td>
      <td>5.48</td>
      <td>31.81</td>
      <td>64.66</td>
      <td>0.80</td>
      <td style="text-decoration: underline;">30.84</td>
      <td style="text-decoration: underline;">28.84</td>
    </tr>
    <tr>
      <td>DOCX-Chain</td>
      <td>30.86</td>
      <td>11.71</td>
      <td>39.62</td>
      <td style="text-decoration: underline;">19.23</td>
      <td style="text-decoration: underline;">10.67</td>
      <td>23.00</td>
      <td>41.60</td>
      <td style="text-decoration: underline;">1.80</td>
      <td>16.96</td>
      <td>21.27</td>
    </tr>
    <tr>
      <td style="font-weight: bold;">DocLayout-YOLO</td>
      <td style="font-weight: bold;">43.71</td>
      <td style="font-weight: bold;">48.71</td>
      <td style="font-weight: bold;">72.83</td>
      <td style="font-weight: bold;">42.67</td>
      <td style="font-weight: bold;">35.40</td>
      <td style="font-weight: bold;">51.44</td>
      <td style="font-weight: bold;">66.84</td>
      <td style="font-weight: bold;">9.54</td>
      <td style="font-weight: bold;">57.54</td>
      <td style="font-weight: bold;">48.71</td>
    </tr>
  </tbody>
</table>


Layout检测config文件参考[layout_detection](./configs/layout_detection.yaml)，数据格式参考[detection_prediction](./demo_data/detection/detection_prediction.json)。

<details>
  <summary>layout_detection.yaml的字段解释</summary>

以下我们以精简格式为例进行展示。`layout_detection.yaml`的配置文件如下：

```YAML
detection_eval:   # 指定task名称，所有的检测相关的任务通用此task
  metrics:
    - COCODet     # 检测任务相关指标，主要是mAP, mAR等
  dataset: 
    dataset_name: detection_dataset_simple_format       # 数据集名称，如果按照规定的输入格式则不需要修改
    ground_truth:
      data_path: ./demo_data/omnidocbench_demo/OmniDocBench_demo.json               # OmniDocBench的JSON文件路径
    prediction:
      data_path: ./demo_data/detection/detection_prediction.json                    # 模型预测结果JSON文件路径
    filter:                                             # 页面级别的筛选
      data_source: exam_paper                           # 需要评测的页面属性以及对应标签
  categories:
    eval_cat:                # 参与最终评测的类别
      block_level:           # block级别的类别，详细类别信息请参考OmniDocBench的评测集介绍部分
        - title              # Title
        - text               # Text
        - abandon            # Includes headers, footers, page numbers, and page annotations
        - figure             # Image
        - figure_caption     # Image caption
        - table              # Table
        - table_caption      # Table caption
        - table_footnote     # Table footnote
        - isolate_formula    # Display formula (this is a layout display formula, lower priority than 14)
        - formula_caption    # Display formula label
    gt_cat_mapping:          # ground truth到最终评测类别的映射表，key是ground truth类别，value是最终评测类别名称
      figure_footnote: figure_footnote
      figure_caption: figure_caption 
      page_number: abandon 
      header: abandon 
      page_footnote: abandon
      table_footnote: table_footnote 
      code_txt: figure 
      equation_caption: formula_caption 
      equation_isolated: isolate_formula
      table: table 
      refernece: text 
      table_caption: table_caption 
      figure: figure 
      title: title 
      text_block: text 
      footer: abandon
    pred_cat_mapping:       # prediction到最终评测类别的映射表，key是prediction类别，value是最终评测类别名称
      title : title
      plain text: text
      abandon: abandon
      figure: figure
      figure_caption: figure_caption
      table: table
      table_caption: table_caption
      table_footnote: table_footnote
      isolate_formula: isolate_formula
      formula_caption: formula_caption
```

使用filter字段可以对数据集进行筛选，比如将`dataset`下设置`filter`字段为`data_source: exam_paper`即筛选数据类型为exam_paper的页面。更多页面属性请参考“评测集介绍”部分。如果希望全量评测，请注释掉`filter`相关字段。

`dataset`部分`prediction`的`data_path`中传入的是模型的prediction，其数据格式为：

```JSON
{
    "results": [
        {
            "image_name": "docstructbench_llm-raw-scihub-o.O-adsc.201190003.pdf_6",                     // 图片名
            "bbox": [53.892921447753906, 909.8675537109375, 808.5555419921875, 1006.2714233398438],     // bounding box信息，分别是左上角和右下角的x,y坐标
            "category_id": 1,                                                                           // 类别序号名称
            "score": 0.9446213841438293                                                                 // 置信度
        }, 
        ...                                                                                             // 所有的bounding box都直接平铺在一个list内部
    ],
    "categories": {"0": "title", "1": "plain text", "2": "abandon", ...}                                // 每个类别序号所对应的类别名称
```

</details>

### 公式检测

OmniDocBench包含每个PDF页面的公式的bounding box信息，因此可以作为Layout检测任务评测的benchmark。

公式检测与Layout检测的格式基本一致。公式包含行内公式和行间公式。在本节提供一个config样例，可以同时评测行间公式和行内公式的检测结果。公式检测可以参考[formula_detection](./configs/formula_detection.yaml)进行配置。

<details>
  <summary>formula_detection.yaml的字段解释</summary>

`formula_detection.yaml`的配置文件如下：

```YAML
detection_eval:   # 指定task名称，所有的检测相关的任务通用此task
  metrics:
    - COCODet     # 检测任务相关指标，主要是mAP, mAR等
  dataset: 
    dataset_name: detection_dataset_simple_format       # 数据集名称，如果按照规定的输入格式则不需要修改
    ground_truth:
      data_path: ./demo_data/omnidocbench_demo/OmniDocBench_demo.json               # OmniDocBench的JSON文件路径
    prediction:
      data_path: ./demo_data/detection/detection_prediction.json                     # 模型预测结果JSON文件路径
    filter:                                             # 页面级别的筛选
      data_source: exam_paper                           # 需要评测的页面属性以及对应标签
  categories:
    eval_cat:                                  # 参与最终评测的类别
      block_level:                             # block级别的类别，详细类别信息请参考OmniDocBench的评测集介绍部分
        - isolate_formula                      # 行间公式
      span_level:                              # span级别的类别，详细类别信息请参考OmniDocBench的评测集介绍部分
        - inline_formula                       # 行内公式
    gt_cat_mapping:                            # ground truth到最终评测类别的映射表，key是ground truth类别，value是最终评测类别名称
      equation_isolated: isolate_formula
      equation_inline: inline_formula
    pred_cat_mapping:                          # prediction到最终评测类别的映射表，key是prediction类别，value是最终评测类别名称
      interline_formula: isolate_formula
      inline_formula: inline_formula
```

config中参数解释以及数据集格式请参考`Layout检测`小节，公式检测与Layout检测小节的主要区别是，在参与最终评测的类别`eval_cat`下新增了`span_level`的类别`inline_formula`，span_level的类别和block_level级别的类别在评测的时候将会共同参与评测。

</details>

## Tools

我们在`tools`目录下提供了一些工具：
- [json2md](./tools/json2md.py) 用于将JSON格式的OmniDocBench转换为Markdown格式；
- [visualization](./tools/visualization.py) 用于可视化OmniDocBench的JSON文件；
- [model_infer](./tools/model_infer)文件夹下提供了一些模型推理的脚本供参考，请在配置了模型环境后使用，包括：
  - `<model_name>_img2md.py` 用于调用模型将图片转换为Markdown格式；
  - `<model_name>_ocr.py` 用于调用模型对block级别的文档文本段落进行文本识别；
  - `<model_name>_formula.py`用于调用模型对行间公式进行公式识别；

## The Evaluation Model Information

### end2end
<table>
  <thead>
    <tr>
      <th>Model Name</th>
      <th>Official Website</th>
      <th>Evaluation Version/Model Weights</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>MinerU</td>
      <td><a href="https://mineru.org.cn/">MinerU</a></td>
      <td>0.9.3</td>
    </tr>
    <tr>
      <td>Marker</td>
      <td><a href="https://github.com/VikParuchuri/marker">Marker</a></td>
      <td>1.2.3</td>
    </tr>
    <tr>
      <td>Mathpix</td>
      <td><a href="https://mathpix.com/">Mathpix</a></td>
      <td>———</td>
    </tr>
    <tr>
      <td>Docling</td>
      <td><a href="https://ds4sd.github.io/docling/">Docling</a></td>
      <td>2.14.0</td>
    </tr>
    <tr>
      <td>GOT-OCR</td>
      <td><a href="https://github.com/Ucas-HaoranWei/GOT-OCR2.0">GOT-OCR GitHub</a></td>
      <td><a href="https://huggingface.co/stepfun-ai/GOT-OCR2_0">Hugging Face GOT-OCR2_0</a></td>
    </tr>
    <tr>
      <td>Nougat</td>
      <td><a href="https://github.com/facebookresearch/nougat">Nougat GitHub</a></td>
      <td><a href="https://huggingface.co/docs/transformers/main/en/model_doc/nougat">Hugging Face Nougat</a></td>
    </tr>
    <tr>
      <td>GPT4o</td>
      <td><a href="https://openai.com/index/hello-gpt-4o/">OpenAI GPT4o</a></td>
      <td>2024-08-06</td>
    </tr>
    <tr>
      <td>Qwen2-VL-72B</td>
      <td><a href="https://qwenlm.github.io/zh/blog/qwen2-vl/">Qwen2-VL Blog</a></td>
      <td><a href="https://huggingface.co/Qwen/Qwen2-VL-72B-Instruct">Hugging Face Qwen2-VL-72B-Instruct</a></td>
    </tr>
    <tr>
      <td>InternVL2-Llama3-76B</td>
      <td><a href="https://github.com/OpenGVLab/InternVL">InternVL GitHub</a></td>
      <td><a href="https://huggingface.co/OpenGVLab/InternVL2-Llama3-76B">Hugging Face InternVL2-Llama3-76B</a></td>
    </tr>
    <tr>
      <td>Paddle</td>
      <td><a href="https://www.paddlepaddle.org.cn/hub/scene/ocr">PaddlePaddle OCR</a></td>
      <td>2.9.1</td>
    </tr>
    <tr>
      <td>Tesseract</td>
      <td><a href="https://tesseract-ocr.github.io/tessdoc/">Tesseract OCR</a></td>
      <td>5.5</td>
    </tr>
    <tr>
      <td>OpenOCR</td>
      <td><a href="https://github.com/Topdu/OpenOCR">OpenOCR GitHub</a></td>
      <td>0.0.6</td>
    </tr>
    <tr>
      <td>EasyOCR</td>
      <td><a href="https://www.easyproject.cn/easyocr/">EasyOCR</a></td>
      <td>1.7.2</td>
    </tr>
    <tr>
      <td>Surya</td>
      <td><a href="https://github.com/VikParuchuri/surya">Surya GitHub</a></td>
      <td>0.5.0</td>
    </tr>
    <tr>
      <td>Pix2tex</td>
      <td><a href="https://p2t.breezedeus.com/">Pix2tex</a></td>
      <td>0.1.2</td>
    </tr>
  </tbody>
</table>

### Layout
<table>
  <thead>
    <tr>
      <th>Model Name</th>
      <th>Official Website</th>
      <th>Evaluation Version/Model Weights</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>DiT-L</td>
      <td><a href="https://github.com/facebookresearch/DiT">DiT-L</a></td>
      <td><a href="https://huggingface.co/docs/transformers/model_doc/dit">Hugging Face DiT</a></td>
    </tr>
    <tr>
      <td>LayoutMv3</td>
      <td><a href="https://github.com/microsoft/unilm/tree/master/layoutlmv3">LayoutMv3</a></td>
      <td><a href="https://huggingface.co/docs/transformers/model_doc/layoutlmv3">Hugging Face LayoutMv3</a></td>
    </tr>
    <tr>
      <td>DOCX-Chain</td>
      <td><a href="https://github.com/AlibabaResearch/AdvancedLiterateMachinery/tree/main/Applications/DocXChain">DOCX-Chain</a></td>
      <td><a href="https://github.com/AlibabaResearch/AdvancedLiterateMachinery/releases/download/v1.2.0-docX-release/DocXLayout_231012.pth">DocXLayout_231012.pth</a></td>
    </tr>
    <tr>
      <td>DocLayout-YOLO</td>
      <td><a href="https://github.com/opendatalab/DocLayout-YOLO">DocLayout-YOLO</a></td>
      <td><a href="https://huggingface.co/spaces/opendatalab/DocLayout-YOLO">Hugging Face DocLayout-YOLO</a></td>
    </tr>
  </tbody>
</table>

### Formula
<table>
  <thead>
    <tr>
      <th>Model Name</th>
      <th>Official Website</th>
      <th>Evaluation Version/Model Weights</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GOT_OCR</td>
      <td><a href="https://github.com/Ucas-HaoranWei/GOT-OCR2.0">GOT_OCR</a></td>
      <td><a href="https://huggingface.co/stepfun-ai/GOT-OCR2_0">Hugging Face GOT-OCR2_0</a></td>
    </tr>
    <tr>
      <td>Mathpix</td>
      <td><a href="https://mathpix.com/">Mathpix</a></td>
      <td>———</td>
    </tr>
    <tr>
      <td>Pix2Tex</td>
      <td><a href="https://p2t.breezedeus.com/">Pix2Tex</a></td>
      <td>0.1.2</td>
    </tr>
    <tr>
      <td>UniMERNet-B</td>
      <td><a href="https://github.com/opendatalab/UniMERNet">UniMERNet-B</a></td>
      <td><a href="https://huggingface.co/datasets/wanderkid/UniMER_Dataset">Hugging Face UniMERNet-B</a></td>
    </tr>
    <tr>
      <td>GPT4o</td>
      <td><a href="https://openai.com/index/hello-gpt-4o/">GPT4o</a></td>
      <td>2024-08-06</td>
    </tr>
    <tr>
      <td>InternVL2-Llama3-76B</td>
      <td><a href="https://github.com/OpenGVLab/InternVL">InternVL2-Llama3-76B</a></td>
      <td><a href="https://huggingface.co/OpenGVLab/InternVL2-Llama3-76B">Huggingface Face InternVL2-Llama3-76B</a></td>
    </tr>
    <tr>
      <td>Qwen2-VL-72B</td>
      <td><a href="https://qwenlm.github.io/zh/blog/qwen2-vl/">Qwen2-VL-72B</a></td>
      <td><a href="https://huggingface.co/Qwen/Qwen2-VL-72B-Instruct">Hugging Face Qwen2-VL-72B-Instruct</a></td>
    </tr>
  </tbody>
</table>

### Table
<table>
  <thead>
    <tr>
      <th>Model Name</th>
      <th>Official Website</th>
      <th>Evaluation Version/Model Weights</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>PaddleOCR</td>
      <td><a href="https://github.com/PaddlePaddle/PaddleOCR">PaddleOCR</a></td>
      <td><a href="https://paddlepaddle.github.io/PaddleOCR/latest/model/index.html">PaddleOCR</a></td>
    </tr>
    <tr>
      <td>RapidTable</td>
      <td><a href="https://github.com/RapidAI/RapidTable">RapidTable</a></td>
      <td><a href="https://www.modelscope.cn/models/RapidAI/RapidTable/files">ModelScope RapidTable</a></td>
    </tr>
    <tr>
      <td>StructEqTable</td>
      <td><a href="https://github.com/Alpha-Innovator/StructEqTable-Deploy/blob/main/README.md">StructEqTable</a></td>
      <td><a href="https://huggingface.co/U4R/StructTable-base">Hugging Face StructEqTable</a></td>
    </tr>
    <tr>
      <td>GOT-OCR</td>
      <td><a href="https://github.com/Ucas-HaoranWei/GOT-OCR2.0">GOT-OCR</a></td>
      <td><a href="https://huggingface.co/stepfun-ai/GOT-OCR2_0">Hugging Face GOT-OCR</a></td>
    </tr>
    <tr>
      <td>Qwen2-VL-7B</td>
      <td><a href="https://github.com/QwenLM/Qwen2-VL">Qwen2-VL-7B</a></td>
      <td><a href="https://huggingface.co/Qwen/Qwen2-VL-7B-Instruct">Hugging Face Qwen2-VL-7B-Instruct</a></td>
    </tr>
    <tr>
      <td>InternVL2-8B</td>
      <td><a href="https://github.com/OpenGVLab/InternVL">InternVL2-8B</a></td>
      <td><a href="https://huggingface.co/OpenGVLab/InternVL2-8B">Hugging Face InternVL2-8B</a></td>
    </tr>
  </tbody>
</table>

## TODO

- [ ] 匹配算法`match_full`接入
- [ ] 针对模型特定输出格式进行匹配后处理优化
- [ ] 增加特殊字符的Unicode映射表

## Known Issues

- 部分模型偶尔会出现输出格式不规范（比如将多栏文本识别为表格，将公式识别为Unicode文本），导致匹配失败的情况，可以针对模型输出格式进行后处理优化
- 由于各种模型对符号的识别能力不同，导致部分符号的识别结果不一致（比如列表的标识符等），目前的文本评测中仅保留中英文本参与评测，后续将增加Unicode映射表来优化

欢迎大家使用OmniDocBench数据集，并提出宝贵的意见和建议，帮助我们不断优化数据集质量和评测工具。有任何意见和建议，欢迎提issue，我们将在第一时间响应。如有评测方案优化可以提PR，我们也将及时review和更新。

## Acknowledgement

- 感谢[Abaka AI](https://abaka.ai)提供的高质量数据集标注
- [PubTabNet](https://github.com/ibm-aur-nlp/PubTabNet) TEDS指标计算
- [latexml](https://github.com/brucemiller/LaTeXML) LaTeX to HTML转换工具
- [Tester](https://github.com/intsig-textin/markdown_tester) Markdown表格转HTML工具

## 版权声明
  
PDF来源从网络公开渠道收集以及社群用户贡献，已剔除了不允许分发的内容，只用作科研，不作为商业用途。若有侵权请联系OpenDataLab@pjlab.org.cn。


## 引用

```bibtex
@misc{ouyang2024omnidocbenchbenchmarkingdiversepdf,
      title={OmniDocBench: Benchmarking Diverse PDF Document Parsing with Comprehensive Annotations}, 
      author={Linke Ouyang and Yuan Qu and Hongbin Zhou and Jiawei Zhu and Rui Zhang and Qunshu Lin and Bin Wang and Zhiyuan Zhao and Man Jiang and Xiaomeng Zhao and Jin Shi and Fan Wu and Pei Chu and Minghao Liu and Zhenxiang Li and Chao Xu and Bo Zhang and Botian Shi and Zhongying Tu and Conghui He},
      year={2024},
      eprint={2412.07626},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2412.07626}, 
}
```
