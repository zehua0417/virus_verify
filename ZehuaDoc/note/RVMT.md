# Expansion of the global RNA virome reveals diverse clades of bacteriophages
**主体部分**:

1. **RNA病毒多样性的显著增加：** 研究显示，RNA病毒的多样性比之前已知的增加了约5倍，特别是在RNA噬菌体方面。
2. **RNA噬菌体在RNA病毒组中的重要角色：** 发现的RNA噬菌体类群在RNA病毒组中占据了相当大的比例，表明它们在微生物生态系统中可能扮演着重要角色。
3. **病毒进化的新见解：** 研究提供了关于RNA病毒，特别是RNA噬菌体进化的新见解，包括它们与宿主的相互作用和可能的宿主转移事件。
4. **病毒分类的更新：** 研究结果支持了RNA病毒Orthornavirae的分类，并对现有的病毒分类体系进行了轻微的调整。

---

## [discovery pipeline](https://github.com/zehua0417/RVMT/tree/main/Discovery_pipeline)

**流程**:

1. **DNA实体序列过滤：** 通过将元转录组contigs与多样的DNA基因组和元基因组进行比较，首先过滤掉可能由DNA实体编码的序列。

2. **RdRP搜索：** 对经过DNA实体序列过滤的序列集合进行迭代搜索RNA依赖RNA聚合酶（RdRP），并将确定的匹配视为推测的RNA病毒。

3. **进一步分析：** 对含有足够完整RdRP的contigs进行进一步分析，以确定其在全球RNA病毒组成中的角色。

4. **元转录组contig相似性筛选：** 使用RdRP-encoding contigs作为诱饵，识别其他元转录组contigs，其与RdRP-encoding contigs具有高度的核酸相似性。

5. **RNA病毒contig鉴定：** 识别并补充已发表来源的RNA病毒contigs，形成一个庞大的RNA病毒数据集。

6. **数据整理和分组：** 对RNA病毒contigs进行去重和分组，形成具有90%平均核苷酸同源性的聚类（RvANI90），进一步展示全球RNA病毒组成的多样性。

7. **全球RNA病毒多样性评估：** 通过分析RNA病毒序列聚类的大小分布和累积曲线，评估全球RNA病毒的多样性，特别关注土壤环境中的丰富度。

8. **变异和特殊性分析：** 对RdRP编码的contigs进行额外的分析，包括检测利用替代遗传密码的迹象以及RdRP内保守基序的洗牌。

**文件结构**:

| dir                    | 文件夹                 | 作用                                                         |
| ---------------------- | ---------------------- | ------------------------------------------------------------ |
| Primary_filtration/    | 一次过滤               | 通过将元转录组contigs与多样的DNA基因组和元基因组进行比较，首先过滤掉可能由DNA实体编码的序列。 |
| RDRP_search/           | 搜索RDRP               | 对经过DNA实体序列过滤的序列集合进行迭代搜索RNA依赖RNA聚合酶（RdRP），并将确定的匹配视为推测的RNA病毒。 |
| Config_set_Enrichment/ | 富集元转录组contig集合 | 使用RDRP-encoding contigs作为诱饵，识别其他元转录组contigs，其与RDRP-encoding contigs具有高度的核酸相似性。 |
| Secondary_filtration/  | 二级过滤               |                                                              |
| Misc/                  | 杂项                   |                                                              |

**mmseqs2**:

* 简单工作流(使用 FASTA/FASTQ 等常见文件格式来简化任务)

  * 简易搜索(使用 FASTA/FASTQ 文件针对另一个 FASTA/FASTQ 文件或预构建的 MMseqs2 目标数据库进行搜索)

  ```shell
  mmseqs easy-search examples/QUERY.fasta examples/DB.fasta alnResult.m8 tmp
  ```

  * 简易聚类(使用级联聚类算法对 FASTA/FASTQ 文件中的条目进行聚类)

  ```shell
  mmseqs easy-cluster examples/DB.fasta clusterRes tmp
  mmseqs easy-linclust examples/DB.fasta clusterRes tmp # 与easy-cluster类似但是适合高效处理非常大的数据集
  ```

* 使用 MMseqs2 工作流程和模块进行搜索(对于更专业的操作, 如: 转换、过滤和外部程序执行, 需要与MMseqs2内部数据库配合使用)

  * 转换数据库(将查询序列与目标序列的FSATA文件转换为序列DB)

  ```shell
  mmseqs createdb examples/QUERY.fasta qdb/queryDB
  mmseqs createdb examples/DB.fasta tdb/targetDB
  ```
  
  * 计算索引(为targetDB计算索引文件以进行快速读入)
  
  ```shell
  mmseqs createindex tdb/targetDB tdb/tmp
  ```
  
  * 搜索(分为两个过程: 预过滤(prefilter)和比对(alignment)默认搜索仅计算分数, 使用-a参数更改比对信息, -s更改搜索速度与灵敏度, --format-output更改输出形式)
  
  ```shell
  mmseqs search qdb/queryDB tdb/targetDB result/resultDB result/tmp
  ```
  
  * 将结果转换为 BLAST 格式(-m 8为传统BLAST, -outfmt 6为BLAST+)
  
  ```shell
  mmseqs convertalis qdb/queryDB tdb/targetDB result/resultDB result/resultDB.m8
  ```
  
  1. 列(1,2): 查询和目标序列/轮廓的标识符。
  2. 列(3): 序列相同性。
  3. 列(4): 比对的长度。
  4. 列(5): 不匹配的数量。
  5. 列(6): 缺口的数量。
  6. 列(7-8, 9-10): 在查询和目标中，领域的起始和结束位置。
  7. 列(11): E值（期望值）。
  8. 列(12): 比特分数。
  
* 使用MMseqs2 工作流程和模块进行聚类

  * 将 FASTA 数据库转换为 MMseqs2 数据库 (DB) 格式

  ```shell
  mmseqs createdb examples/DB.fasta db/DB
  ```

  * 运行聚类

  ```shell
  mmseqs cluster db/DB cluster/DB_clu tmp
  ```

  * 转换为tsv格式
  
  ```shell
  mmseqs createtsv db/DB cluster/DB cluster/DB_clu.tsv
  ```
  
  * 添加序列信息和生成结果
  
  ```shell
  mmseqs createseqfiledb db/DB cluster/DB_clu DB_clu_seq # 将序列信息添加到结果数据库
  mmseqs result2flat DB DB DB_clu_seq DB_clu_seq.fasta # 生成结果的flat文件
  ```
  
  