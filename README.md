
# ErmineR <img src="ermineR.png" align="right" height="100px"/>

[![Build
Status](https://travis-ci.org/PavlidisLab/ermineR.svg?branch=master)](https://travis-ci.org/PavlidisLab/ermineR)
[![codecov](https://codecov.io/gh/PavlidisLab/ermineR/branch/master/graph/badge.svg)](https://codecov.io/gh/PavlidisLab/ermineR)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/PavlidisLab/ermineR?branch=master&svg=true)](https://ci.appveyor.com/project/PavlidisLab/ermineR)

This is an R wrapper for Pavlidis Lab’s
[ermineJ](http://erminej.msl.ubc.ca/). A tool for gene set enrichment
analysis with multifunctionality correction.

# Table of Contents

  - [Installation](#installation)
  - [Usage](#usage)
      - [Replicable go terms](#replicable-go-terms)
      - [Annotations](#annotations)
      - [Examples](#examples)
          - [Use GSR with gene scores](#use-gsr-with-gene-scores)
          - [Use Precision Recall with gene
            scores](#use-precision-recall-with-gene-scores)
          - [Use ORA with a hitlist](#use-ora-with-a-hitlist)
          - [Using your own GO
            annotations](#using-your-own-go-annotations)

## Installation

ermineR requries 64 bit version of java to function. If you are a Mac
user make sure you have the java SDK.

After java is installed you can install ermineR by doing

``` r
devtools::install_github('PavlidisLab/ermineR')
```

If ermineR cannot find your java home by itself. Use either install
rJava or use `Sys.setenv(JAVA_HOME=javaHome)` to point ermineR to the
right path.

Some users report that the ermineJ executable loses its exection
privilage after installation. If this happens you will get an error
    like

    "Error in (function (annotation = NULL, aspects = c("Molecular Function",  :
     Something went wrong. Blame the dev
    sh: [library installation path]/ermineR/ermineJ-3.1.2/bin/ermineJ.sh: Permission denied "

To fix this just
    do

    chmod +x [library installation path]/ermineR/ermineJ-3.1.2/bin/ermineJ.sh

You may need `sudo` depending on where you install your packages

## Usage

See documentation for `ora`, `roc`, `gsr`, `precRecall` and `corr` to
see how to use them.

An explanation of what each method does is given. We recommend users
start with the `precRecall` (for gene ranking-based enrichment analysis)
or `ora` (for hit-list over-representation analysis).

### Replicable go terms

GO terms are updated frequently so results [can differ between
versions](https://gotrack.msl.ubc.ca/). The default option of all
ermineR functions is to get the latest GO version however this means you
may get different results when you repeat the experiment later. If you
want to use a specific version of GO, ermineR provides functions to deal
with that.

  - `goToday`: Downloads the latest version of go to a path you provide
  - `getGoDates`: Lists all dates where a go version is available, from
    the most recent to oldest
  - `goAtDate`: Given a valid date, downloads the Go version from a
    specific date to a file path you provide

To use a specific version of GO, make sure to set `geneSetDescription`
argument of all ermineR functions to the file path where you saved the
go terms

### Annotations

ErmineR requires annotation files to work. These files include gene
identifiers and their Go annotations, along with some optional
information. By default, ermineR supports annotation files generated by
[Gemma](https://gemma.msl.ubc.ca/home.html). And will automatically
download them if you provide a valid annotation name. You can get a list
of valid annotation names using `listGemmaAnnotations()`. As a general
rule, if your platform has an identifier in GEO, the identifier that
starts with “GPL” is used as the Gemma identifier as well. There are
also generic annotation files available that contain all genes from a
species. These are typically named something like “Generic\_human”.

You can manually download these annotation files from
<https://gemma.msl.ubc.ca/annots/> or by using the `getGemmaAnnot`
function. ErmineR typically uses “noParents” versions of these files
since parent terms are derived using the ontology file acquired from GO.

### Examples

#### Use GSR with gene scores

Here we will use a mock scores file located in our tests directory. The
score file is specifically created to be enriched in genes with the term
<GO:0051082>.

``` r
scores = read.table("tests/testthat/testFiles/pValues", header=T, row.names = 1)
head(scores)
```

    ##                pvalue
    ## 206190_at   0.3163401
    ## 208385_at   0.5186824
    ## 65086_at    0.6620389
    ## 202281_at   0.4068895
    ## 211622_s_at 0.9128846
    ## 219257_s_at 0.2936740

This scores file only includes scores for 118 genes. The file was
generated using GPL96’s probesets so that is the annotation we’ll be
using. Any gene that is not reperesented by the score file will be
ignored.

``` r
gsrOut = gsr(annotation = 'GPL96',
                 scores = scores,
                 scoreColumn = 1,
                 iterations = 10000,
                 bigIsBetter = FALSE,
                 logTrans = TRUE)

head(gsrOut$results) %>% knitr::kable()
```

| Name                                            | ID           | NumProbes | NumGenes | RawScore |  Pval | CorrectedPvalue | MFPvalue | CorrectedMFPvalue | Multifunctionality | Same as | GeneMembers                                                                                                                                                                               |
| :---------------------------------------------- | :----------- | --------: | -------: | -------: | ----: | --------------: | -------: | ----------------: | -----------------: | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| protein folding                                 | <GO:0006457> |        42 |       24 | 3.084202 | 0e+00 |        0.000000 |    0e+00 |          0.000000 |              0.261 | NA      | AIP|CALR|CCT5|CCT6A|CDC37L1|CLGN|CLPX|DNAJB1|DNAJB4|GAK|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|NUDC|PFDN6|PTGES3|RUVBL2|SPHK1|ST13|TCP1|TOR1A|UGGT1|                                           |
| cellular component assembly                     | <GO:0022607> |        64 |       29 | 2.384466 | 0e+00 |        0.000000 |    4e-04 |          0.010480 |              0.822 | NA      | ARC|CALR|CHAF1A|CLGN|CPT1A|EPHB2|GAK|GEMIN2|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPD1|HTRA2|PFDN6|PIKFYVE|PRKCI|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UNC13B|ZNF207|      |
| cellular component biogenesis                   | <GO:0044085> |        65 |       30 | 2.420182 | 0e+00 |        0.000000 |    2e-04 |          0.006550 |              0.798 | NA      | AAMP|ARC|CALR|CHAF1A|CLGN|CPT1A|EPHB2|GAK|GEMIN2|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPD1|HTRA2|PFDN6|PIKFYVE|PRKCI|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UNC13B|ZNF207| |
| unfolded protein binding                        | <GO:0051082> |        56 |       30 | 3.305791 | 0e+00 |        0.000000 |    0e+00 |          0.000000 |              0.238 | NA      | AAMP|AIP|CALR|CCT5|CCT6A|CDC37L1|CHAF1A|CLGN|CLPX|DNAJB1|DNAJB4|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|NUDC|PFDN6|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UGGT1|   |
| protein-containing complex assembly             | <GO:0065003> |        49 |       23 | 2.542674 | 1e-04 |        0.002620 |    1e-04 |          0.004367 |              0.627 | NA      | ARC|CALR|CHAF1A|CLGN|CPT1A|GEMIN2|HMGCR|HSP90AA1|HSPA1A|HSPD1|HTRA2|PFDN6|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|UNC13B|ZNF207|                                           |
| protein-containing complex subunit organization | <GO:0043933> |        51 |       24 | 2.463916 | 2e-04 |        0.004367 |    4e-04 |          0.008733 |              0.604 | NA      | ARC|CALR|CHAF1A|CLGN|CPT1A|GAK|GEMIN2|HMGCR|HSP90AA1|HSPA1A|HSPD1|HTRA2|PFDN6|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|UNC13B|ZNF207|                                       |

#### Use Precision Recall with gene scores

We will use the same scores file from the example above

``` r
precRecallOut = precRecall(annotation = 'GPL96',
                           scores = scores,
                           scoreColumn = 1,
                           iterations = 10000,
                           bigIsBetter = FALSE,
                           logTrans = TRUE)

head(precRecallOut$results) %>% knitr::kable()
```

| Name                     | ID           | NumProbes | NumGenes |  RawScore |  Pval | CorrectedPvalue | MFPvalue | CorrectedMFPvalue | Multifunctionality | Same as                                                      | GeneMembers                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :----------------------- | :----------- | --------: | -------: | --------: | ----: | --------------: | -------: | ----------------: | -----------------: | :----------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| intracellular            | <GO:0005622> |       130 |       71 | 0.9633054 | 0e+00 |         0.00000 |    1e-04 |          0.004367 |              0.519 | [GO:0044424|intracellular](GO:0044424%7Cintracellular) part, | AAMP|AIP|ARC|ARF3|BHMT2|CALR|CCNG1|CCT5|CCT6A|CCT8L2|CDC37L1|CHAF1A|CLGN|CLPX|CPT1A|CRABP1|DDX46|DMBT1|DNAJB1|DNAJB4|DZIP3|EPHB2|FOXB1|FRS2|GAK|GEMIN2|HCK|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|ITIH2|LIPF|MAN1B1|NELFA|NR2E3|NUDC|OGG1|PASK|PEX5|PFDN6|PIKFYVE|PLCH1|POLR3K|PPARA|PRKCI|PRPSAP1|PTGES3|RUVBL2|SEMA3B|SHOX2|SHQ1|SPHK1|SRSF10|ST13|SULF1|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UGGT1|UNC13B|USP33|VPS8|YIPF2|ZCCHC8|ZNF207| |
| protein folding          | <GO:0006457> |        42 |       24 | 0.7037590 | 0e+00 |         0.00000 |    0e+00 |          0.000000 |              0.261 | NA                                                           | AIP|CALR|CCT5|CCT6A|CDC37L1|CLGN|CLPX|DNAJB1|DNAJB4|GAK|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|NUDC|PFDN6|PTGES3|RUVBL2|SPHK1|ST13|TCP1|TOR1A|UGGT1|                                                                                                                                                                                                                                                                                               |
| intracellular part       | <GO:0044424> |       130 |       71 | 0.9633054 | 0e+00 |         0.00000 |    1e-04 |          0.004367 |              0.520 | [GO:0005622|intracellular](GO:0005622%7Cintracellular),      | AAMP|AIP|ARC|ARF3|BHMT2|CALR|CCNG1|CCT5|CCT6A|CCT8L2|CDC37L1|CHAF1A|CLGN|CLPX|CPT1A|CRABP1|DDX46|DMBT1|DNAJB1|DNAJB4|DZIP3|EPHB2|FOXB1|FRS2|GAK|GEMIN2|HCK|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|ITIH2|LIPF|MAN1B1|NELFA|NR2E3|NUDC|OGG1|PASK|PEX5|PFDN6|PIKFYVE|PLCH1|POLR3K|PPARA|PRKCI|PRPSAP1|PTGES3|RUVBL2|SEMA3B|SHOX2|SHQ1|SPHK1|SRSF10|ST13|SULF1|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UGGT1|UNC13B|USP33|VPS8|YIPF2|ZCCHC8|ZNF207| |
| unfolded protein binding | <GO:0051082> |        56 |       30 | 0.9127721 | 0e+00 |         0.00000 |    0e+00 |          0.000000 |              0.238 | NA                                                           | AAMP|AIP|CALR|CCT5|CCT6A|CDC37L1|CHAF1A|CLGN|CLPX|DNAJB1|DNAJB4|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|NUDC|PFDN6|PTGES3|RUVBL2|SHQ1|SRSF10|ST13|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UGGT1|                                                                                                                                                                                                                                                       |
| protein binding          | <GO:0005515> |       115 |       63 | 0.9147643 | 2e-04 |         0.00655 |    3e-04 |          0.009825 |              0.536 | NA                                                           | AAMP|AIP|C5AR2|CALR|CCNG1|CCT5|CCT6A|CDC37L1|CHAF1A|CLGN|CLPX|CPT1A|CRABP1|DMBT1|DNAJB1|DNAJB4|DZIP3|EPHB2|FOXB1|FRS2|GAK|GEMIN2|GPR17|HCK|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|NELFA|NUDC|OGG1|PASK|PEX5|PFDN6|PIKFYVE|PPARA|PRKCI|PRPSAP1|PTGES3|RUVBL2|SEMA3B|SHQ1|SLC24A1|SPHK1|SRSF10|ST13|TAPBP|TBKBP1|TCP1|TNFRSF12A|TOMM20|TOR1A|TUBB4B|UGGT1|UNC13B|USP33|VPS8|YIPF2|ZCCHC8|ZNF207|                                         |
| cytoplasm                | <GO:0005737> |       118 |       64 | 0.9094862 | 3e-04 |         0.00786 |    8e-04 |          0.020960 |              0.404 | NA                                                           | AAMP|AIP|ARC|ARF3|BHMT2|CALR|CCNG1|CCT5|CCT6A|CCT8L2|CDC37L1|CLGN|CLPX|CPT1A|CRABP1|DMBT1|DNAJB1|DNAJB4|DZIP3|EPHB2|FRS2|GAK|GEMIN2|HCK|HMGCR|HSP90AA1|HSPA1A|HSPA9|HSPB6|HSPD1|HTRA2|ITIH2|LIPF|MAN1B1|NELFA|NUDC|OGG1|PASK|PEX5|PFDN6|PIKFYVE|PLCH1|POLR3K|PRKCI|PRPSAP1|PTGES3|RUVBL2|SEMA3B|SHQ1|SPHK1|SRSF10|ST13|SULF1|TAPBP|TCP1|TOMM20|TOR1A|TUBB4B|UGGT1|UNC13B|USP33|VPS8|YIPF2|ZNF207|                                             |

#### Use ORA with a hitlist

``` r
library(dplyr)


# genes for GO:0051082
hitlist = c("AAMP", "AFG3L2", "AHSP", "AIP", "AIPL1", "APCS", "BBS12", 
            "CALR", "CALR3", "CANX", "CCDC115", "CCT2", "CCT3", "CCT4", "CCT5", 
            "CCT6A", "CCT6B", "CCT7", "CCT8", "CCT8L1P", "CCT8L2", "CDC37", 
            "CDC37L1", "CHAF1A", "CHAF1B", "CLGN", "CLN3", "CLPX", "CRYAA", 
            "CRYAB", "DNAJA1", "DNAJA2", "DNAJA3", "DNAJA4", "DNAJB1", "DNAJB11", 
            "DNAJB13", "DNAJB2", "DNAJB4", "DNAJB5", "DNAJB6", "DNAJB8", 
            "DNAJC4", "DZIP3", "ERLEC1", "ERO1B", "FYCO1", "GRPEL1", "GRPEL2", 
            "GRXCR2", "HEATR3", "HSP90AA1", "HSP90AA2P", "HSP90AA4P", "HSP90AA5P", 
            "HSP90AB1", "HSP90AB2P", "HSP90AB3P", "HSP90AB4P", "HSP90B1", 
            "HSP90B2P", "HSPA1A", "HSPA1B", "HSPA1L", "HSPA2", "HSPA5", "HSPA6", 
            "HSPA8", "HSPA9", "HSPB6", "HSPD1", "HSPE1", "HTRA2", "LMAN1", 
            "MDN1", "MKKS", "NAP1L4", "NDUFAF1", "NPM1", "NUDC", "NUDCD2", 
            "NUDCD3", "PDRG1", "PET100", "PFDN1", "PFDN2", "PFDN4", "PFDN5", 
            "PFDN6", "PIKFYVE", "PPIA", "PPIB", "PTGES3", "RP2", "RUVBL2", 
            "SCAP", "SCG5", "SERPINH1", "SHQ1", "SIL1", "SPG7", "SRSF10", 
            "SRSF12", "ST13", "SYVN1", "TAPBP", "TCP1", "TMEM67", "TOMM20", 
            "TOR1A", "TRAP1", "TTC1", "TUBB4B", "UGGT1", "ZFYVE21")
oraOut = ora(annotation = 'Generic_human',
             hitlist = hitlist)

head(oraOut$results) %>% knitr::kable()
```

| Name                                        | ID           | NumProbes | NumGenes | RawScore | Pval | CorrectedPvalue | MFPvalue | CorrectedMFPvalue | Multifunctionality | Same as | GeneMembers                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| :------------------------------------------ | :----------- | --------: | -------: | -------: | ---: | --------------: | -------: | ----------------: | -----------------: | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unfolded protein binding                    | <GO:0051082> |       129 |      129 |      107 |    0 |               0 |        0 |                 0 |              0.652 | NA      | AAMP|AFG3L2|AHSP|AIP|AIPL1|APCS|CALR|CALR3|CANX|CCDC115|CCT2|CCT3|CCT4|CCT5|CCT6A|CCT6B|CCT7|CCT8|CDC37|CDC37L1|CHAF1A|CHAF1B|CLGN|CLN3|CLPX|CRYAA|CRYAB|DNAJA1|DNAJA2|DNAJA3|DNAJA4|DNAJB1|DNAJB11|DNAJB13|DNAJB2|DNAJB4|DNAJB5|DNAJB6|DNAJB8|DNAJC3|DNAJC4|ERLEC1|ERN1|ERN2|ERO1B|GRPEL1|GRPEL2|HEATR3|HSP90AA1|HSP90AA2P|HSP90AA4P|HSP90AA5P|HSP90AB1|HSP90AB2P|HSP90AB3P|HSP90AB4P|HSP90B1|HSP90B2P|HSPA13|HSPA14|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA5|HSPA6|HSPA7|HSPA8|HSPA9|HSPB6|HSPD1|HSPE1|HTRA2|LMAN1|MDN1|MKKS|NAP1L4|NDUFAF1|NKTR|NPM1|NUDC|NUDCD2|NUDCD3|NWD2|PDRG1|PET100|PFDN1|PFDN2|PFDN4|PFDN5|PFDN6|PPIA|PPIAL4A|PPIAL4C|PPIAL4D|PPIAL4E|PPIAL4F|PPIAL4G|PPIB|PPIC|PPID|PPIE|PPIF|PPIG|PPIH|PPIL6|PTGES3|RP2|RUVBL2|SCAP|SCG5|SERPINH1|SHQ1|SIL1|SPG7|SRSF10|SRSF12|ST13|SYVN1|TAPBP|TCP1|TMEM67|TOMM20|TOR1A|TRAP1|TTC1|TUBB4B|UGGT1|UGGT2| |
| chaperone-mediated protein folding          | <GO:0061077> |        59 |       59 |       24 |    0 |               0 |        0 |                 0 |              0.647 | NA      | BAG1|CALR|CANX|CCT2|CD74|CHORDC1|CLU|CRTAP|CSNK2A1|DFFA|DNAJB1|DNAJB12|DNAJB13|DNAJB14|DNAJB2|DNAJB4|DNAJB5|DNAJB8|DNAJC18|DNAJC24|DNAJC5|DNAJC7|ERO1A|FKBP1A|FKBP1B|FKBP4|FKBP5|GAK|HSPA13|HSPA14|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA5|HSPA6|HSPA7|HSPA8|HSPA9|HSPB1|HSPB6|HSPE1|HSPH1|P3H1|PDIA4|PEX19|PPIB|PPID|PTGES3|SGTA|ST13|ST13P4|ST13P5|TOR1A|TOR1B|TOR2A|TRAP1|UNC45A|UNC45B|                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| chaperone binding                           | <GO:0051087> |       100 |      100 |       26 |    0 |               0 |        0 |                 0 |              0.801 | NA      | AHSA1|AHSA2P|ALB|AMFR|ATP1A1|ATP1A2|ATP1A3|ATP7A|BAG1|BAG2|BAG3|BAG4|BAG5|BAK1|BAX|BIN1|BIRC2|BIRC5|CALR|CDC25A|CDC37|CDC37L1|CDKN1B|CFTR|CLU|CP|CTSC|DNAJA1|DNAJA2|DNAJA4|DNAJB1|DNAJB13|DNAJB2|DNAJB4|DNAJB5|DNAJB6|DNAJB7|DNAJB8|DNAJB9|DNAJC1|DNAJC10|DNAJC3|DNLZ|ERP29|FGB|FICD|FN1|FNIP1|FNIP2|GAK|GET4|GNB5|GRPEL1|GRPEL2|HES1|HSCB|HSPA2|HSPA5|HSPA8|HSPB6|HSPD1|HSPE1|HYOU1|KSR1|LRP2|MAPT|OGDH|PACRG|PDPN|PFDN4|PFDN6|PIH1D3|PLG|PRKN|PRNP|PTGES3|RNF207|SACS|SDF2L1|SLC25A17|SOD1|ST13|STIP1|STUB1|SYVN1|TBCA|TBCC|TBCD|TBCE|TERT|TIMM10|TIMM44|TIMM9|TP53|TSACC|TSC1|UBL4A|USP13|VWF|WRAP53|                                                                                                                                                                                                                                                    |
| ‘de novo’ protein folding                   | <GO:0006458> |        41 |       41 |       19 |    0 |               0 |        0 |                 0 |              0.532 | NA      | BAG1|CCT2|CD74|CHCHD4|DNAJB1|DNAJB12|DNAJB13|DNAJB14|DNAJB4|DNAJB5|DNAJC18|DNAJC2|DNAJC7|ENTPD5|ERO1A|FKBP1A|FKBP1B|GAK|HSPA13|HSPA14|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA5|HSPA6|HSPA7|HSPA8|HSPA9|HSPD1|HSPE1|HSPH1|PTGES3|SELENOF|ST13|ST13P4|ST13P5|TOR1A|TOR1B|TOR2A|UGGT1|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ‘de novo’ posttranslational protein folding | <GO:0051084> |        37 |       37 |       18 |    0 |               0 |        0 |                 0 |              0.413 | NA      | BAG1|CCT2|CD74|CHCHD4|DNAJB1|DNAJB12|DNAJB13|DNAJB14|DNAJB4|DNAJB5|DNAJC18|DNAJC7|ENTPD5|ERO1A|GAK|HSPA13|HSPA14|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA5|HSPA6|HSPA7|HSPA8|HSPA9|HSPE1|HSPH1|PTGES3|SELENOF|ST13|ST13P4|ST13P5|TOR1A|TOR1B|TOR2A|UGGT1|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| chaperone complex                           | <GO:0101031> |        22 |       22 |       15 |    0 |               0 |        0 |                 0 |              0.595 | NA      | BAG2|BAG3|CCT2|CCT3|CCT4|CCT5|CCT6A|CCT6B|CCT7|CCT8|CCT8L1P|CCT8L2|CDC37|HSP90AB1|HSPA8|HSPB8|PPP5C|PTGES3|STIP1|STUB1|TCP1|TSC1|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

#### Using your own GO annotations

If you want to use your own GO annotations instead of getting files
provided by Pavlidis Lab, you can use `makeAnnotation` after turning
your annotations into a list. See the example below

``` r
library('org.Hs.eg.db') # get go terms from bioconductor 
goAnnots = as.list(org.Hs.egGO)
goAnnots = goAnnots %>% lapply(names)
goAnnots %>% head
```

    ## $`1`
    ##  [1] "GO:0002576" "GO:0008150" "GO:0043312" "GO:0005576" "GO:0005576"
    ##  [6] "GO:0005576" "GO:0005615" "GO:0031093" "GO:0034774" "GO:0062023"
    ## [11] "GO:0070062" "GO:0072562" "GO:1904813" "GO:0003674"
    ## 
    ## $`2`
    ##  [1] "GO:0001869" "GO:0002576" "GO:0007597" "GO:0010951" "GO:0022617"
    ##  [6] "GO:0048863" "GO:0051056" "GO:0005576" "GO:0005576" "GO:0005829"
    ## [11] "GO:0031093" "GO:0062023" "GO:0070062" "GO:0072562" "GO:0002020"
    ## [16] "GO:0004867" "GO:0005102" "GO:0005515" "GO:0019838" "GO:0019899"
    ## [21] "GO:0019959" "GO:0019966" "GO:0043120" "GO:0048306"
    ## 
    ## $`3`
    ## NULL
    ## 
    ## $`9`
    ## [1] "GO:0006805" "GO:0005829" "GO:0004060"
    ## 
    ## $`10`
    ## [1] "GO:0006805" "GO:0005829" "GO:0004060" "GO:0005515"
    ## 
    ## $`11`
    ## NULL

The goAnnots object we created has go terms per entrez ID. Similar lists
can be obtained from other species db packages in bioconductor and some
array annotation packages. We will now use the `makeAnnotation` function
to create our annotation file. This file will have the names of this
list (entrez IDs) as gene identifiers so any score or hitlist file you
provide should have the entrez IDs as well.

`makeAnnotation` only needs the list with gene identifiers and go terms
to work. But if you want to have a complete annotation file you can also
provide gene symbols and gene names. Gene names have no effect on the
analysis. Gene symbols matter if you are [providing custom gene
sets](http://erminej.msl.ubc.ca/help/input-files/gene-sets/) and using
“Option 2” or if same genes are represented by multiple gene
identifiers (eg. probes). Gene symbols will also be returned in the
`GeneMembers` column of the output. If they are not provided, gene IDs
will also be used as gene symbols

Here we’ll set them both for good measure.

``` r
geneSymbols = as.list(org.Hs.egSYMBOL) %>% unlist
geneName = as.list(org.Hs.egGENENAME) %>% unlist

annotation = makeAnnotation(goAnnots,
                            symbol = geneSymbols,
                            name = geneName,
                            output = NULL, # you can choose to save the annotation to a file
                            return = TRUE) # if you only want to save it to a file, you don't need to return
```

Now that we have the annotation object, we can use it to run an
analysis. We’ll try to generate a hitlist only composed of genes
annotated with <GO:0051082>.

``` r
mockHitlist = goAnnots %>% sapply(function(x){'GO:0051082' %in% x}) %>% 
    {goAnnots[.]} %>% 
    names

mockHitlist %>% head
```

    ## [1] "325"  "811"  "821"  "871"  "908"  "1047"

``` r
oraOut = ora(annotation = annotation,
             hitlist = mockHitlist)

head(oraOut$results) %>% knitr::kable()
```

| Name                                        | ID           | NumProbes | NumGenes | RawScore | Pval | CorrectedPvalue | MFPvalue | CorrectedMFPvalue | Multifunctionality | Same as | GeneMembers                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------------------------------ | :----------- | --------: | -------: | -------: | ---: | --------------: | -------: | ----------------: | -----------------: | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unfolded protein binding                    | <GO:0051082> |        83 |       83 |       83 |    0 |               0 |        0 |                 0 |              0.680 | NA      | AFG3L2|AHSP|AIP|AIPL1|APCS|CALR|CALR3|CANX|CCT2|CCT3|CCT4|CCT5|CCT6A|CCT6B|CCT7|CCT8|CHAF1A|CHAF1B|CLGN|CLN3|CLPX|CRYAA|CRYAB|DNAJA1|DNAJA2|DNAJA3|DNAJA4|DNAJB1|DNAJB11|DNAJB2|DNAJB6|DNAJB8|DNAJC4|ERLEC1|ERO1B|GRPEL1|HSP90AA5P|HSP90B2P|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA5|HSPA6|HSPA8|HSPA9|HSPB6|HSPD1|HSPE1|HTRA2|LMAN1|MDN1|MKKS|NAP1L4|NDUFAF1|NPM1|PDRG1|PFDN1|PFDN2|PFDN4|PFDN5|PFDN6|PPIA|PTGES3|RP2|RUVBL2|SCAP|SCG5|SERPINH1|SIL1|SPG7|SRSF10|SRSF12|ST13|SYVN1|TAPBP|TCP1|TMEM67|TOMM20|TOR1A|TTC1|TUBB4B|UGGT1|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| protein folding                             | <GO:0006457> |       173 |      173 |       58 |    0 |               0 |        0 |                 0 |              0.891 | NA      | AIP|ALG12|AMFR|APCS|ATF6|B2M|BAG1|BAG2|BAG3|BAG4|BAG5|CALR|CALR3|CANX|CCT2|CCT3|CCT4|CCT5|CCT6A|CCT6B|CCT7|CCT8|CD74|CHCHD4|CHORDC1|CLGN|CLPX|CLU|CRTAP|CRYAB|CSNK2A1|CSNK2A2|CSNK2B|CWC27|DERL1|DFFA|DNAJA1|DNAJA2|DNAJA3|DNAJA4|DNAJB1|DNAJB11|DNAJB12|DNAJB14|DNAJB2|DNAJB6|DNAJB8|DNAJC1|DNAJC10|DNAJC19|DNAJC2|DNAJC21|DNAJC24|DNAJC4|DNAJC5|DNAJC7|ENGASE|ENTPD5|ERO1A|ERO1B|ERP29|ERP44|FKBP1A|FKBP1B|FKBP4|FKBP5|FKBP6|FKBP9|FUT10|GAK|GANAB|GNAI1|GNAI2|GNAI3|GNAO1|GNAT1|GNAT2|GNAT3|GNAZ|GNB1|GNB2|GNB3|GNB4|GNB5|GNG2|GRPEL1|GRPEL2|HSP90AA1|HSP90AA5P|HSP90B1|HSP90B2P|HSPA14|HSPA1A|HSPA1B|HSPA1L|HSPA2|HSPA4L|HSPA5|HSPA6|HSPA8|HSPB1|HSPB6|HSPBP1|HSPD1|HSPE1|HSPH1|LMAN1|LMAN2L|LTBP4|MESD|MKKS|MLEC|MOGS|MPDU1|NFYC|NGLY1|NPPA|NPPB|NPPC|P3H1|PDCL|PDCL3|PDIA2|PDIA3|PDIA4|PDIA5|PDIA6|PDRG1|PEX19|PFDN1|PFDN2|PFDN4|PFDN5|PFDN6|PPIA|PPIB|PPID|PPIH|PPIL1|PPIL2|PPIL3|PRDX4|PRKCSH|PSMC1|PTGES3|RAD23B|RANBP2|RGS7|RGS9|RIC3|RP2|RUVBL2|SACS|SELENOF|SGTA|SIL1|SPHK1|ST13|TBCA|TBCC|TBCD|TBCE|TCP1|TOR1A|TOR1B|TOR2A|TRAP1|TTC1|UBXN1|UGGT1|VBP1|VCP|WFS1| |
| chaperone binding                           | <GO:0051087> |        91 |       91 |       19 |    0 |               0 |        0 |                 0 |              0.913 | NA      | AHSA1|AHSA2P|ALB|AMFR|ATP1A1|ATP1A2|ATP1A3|ATP7A|BAG1|BAG2|BAG3|BAG4|BAG5|BAK1|BAX|BIN1|BIRC2|BIRC5|CALR|CDC25A|CFTR|CLU|CP|CTSC|DNAJA1|DNAJA2|DNAJA4|DNAJB1|DNAJB2|DNAJB4|DNAJB5|DNAJB6|DNAJB7|DNAJB8|DNAJB9|DNAJC1|DNAJC10|ERP29|FGB|FICD|FN1|FNIP1|FNIP2|GAK|GET4|GNB5|GRN|GRPEL1|GRPEL2|HES1|HSPA2|HSPA5|HSPA8|HSPB6|HSPD1|HSPE1|HYOU1|KSR1|LRP2|MAPT|PACRG|PDPN|PFDN4|PFDN6|PIH1D3|PLG|PRKN|RNF207|SACS|SCARB2|SDF2L1|SLC25A17|SOD1|ST13|STIP1|STUB1|SYVN1|TBCA|TBCC|TBCD|TBCE|TERT|TIMM10|TIMM9|TP53|TSACC|TSC1|UBL4A|USP13|VWF|WRAP53|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| protein stabilization                       | <GO:0050821> |       171 |      171 |       16 |    0 |               0 |        0 |                 0 |              0.944 | NA      | A1CF|AAK1|AFM|ANK2|APOA1|APOA2|ATF7IP|ATP1B1|ATP1B2|ATP1B3|BAG2|BAG3|BAG5|BAG6|C10orf90|CALR|CAMLG|CCNH|CCT2|CCT3|CCT4|CCT5|CCT6A|CCT7|CCT8|CDK7|CDKN1A|CHEK2|CHP1|CLU|COG3|COG7|CPN2|CREB1|CREBL2|CRTAP|CRYAA|CRYAB|CSN3|DSG1|DVL1|DVL3|EFNA1|EP300|EPHA4|FBXW7|FKBPL|FLNA|FLOT1|FLOT2|GAPDH|GBA3|GNAQ|GOLGA7|GPIHBP1|GRN|GTPBP4|GTSE1|HCFC1|HIP1|HIP1R|HIST1H1B|HPS4|HSP90AA1|HSP90AB1|HSPA1A|HSPA1B|HSPD1|HYPK|IFT46|IGF1|IRGM|LAMP1|LAMP2|MARCH7|MCM8|MDM4|MIR101-1|MIR101-2|MORC3|MSX1|MT3|MTMR9|MUL1|NAA15|NAA16|NAPG|NCLN|NLK|NOP53|OTUD3|P3H1|PARK7|PDCD10|PER3|PEX19|PEX6|PFN1|PFN2|PHB|PHB2|PIK3R1|PIM1|PIM2|PIN1|PINK1|PLPP3|PPARGC1A|PPIB|PRKCD|PRKN|PRKRA|PTEN|PTGES3|PYHIN1|RASSF2|RPL11|RPL23|RPL5|RPS7|RTN4|SAV1|SAXO1|SEC16A|SEL1L|SMAD3|SMAD7|SMO|SOX17|SOX4|STK3|STK4|STX12|STXBP1|STXBP4|SUMO1|SWSAP1|SYVN1|TAF1|TAF9|TAF9B|TBL1X|TBRG1|TCP1|TELO2|TESC|TMEM88|TNIP2|TP53|TREX1|TRIM44|TSC1|TSPAN1|TYROBP|UBE2B|USP13|USP19|USP2|USP27X|USP33|USP36|USP7|USP9X|VHL|WDR81|WFS1|WIZ|WNT10B|ZBED3|ZNF207|ZSWIM7|                                             |
| heat shock protein binding                  | <GO:0031072> |       108 |      108 |       14 |    0 |               0 |        0 |                 0 |              0.931 | NA      | ADORA1|AHR|AHSA1|AHSA2P|APAF1|APOA1|APOA2|ARNTL|BAG2|BAG6|BAK1|BAX|BCOR|CAMKMT|CDC37|CDK1|CDK5|CDKN1B|CHORDC1|CREB1|CSNK2A1|CYP1A1|CYP2E1|DAXX|DNAJA1|DNAJA2|DNAJA3|DNAJA4|DNAJB1|DNAJB12|DNAJB14|DNAJB2|DNAJB6|DNAJB9|DNAJC10|DNAJC2|DNAJC7|DNAJC8|DNAJC9|EEF1AKMT3|EIF2AK3|ERN1|ETFBKMT|FAF1|FGF1|FICD|FKBP4|FKBP5|FKBP6|GBP1|GPR37|HDAC2|HDAC6|HDAC8|HIF1A|HIKESHI|HSF1|HSP90AB1|HSPA1A|HSPA1B|HSPA1L|HSPA6|HSPA8|HTT|IQCG|IRAK1|ITGAM|ITGB2|KCNJ11|KDR|KPNB1|KSR1|LIMK1|LMAN2|MAPT|METTL18|METTL21A|METTL21C|METTL22|METTL23|MVD|NOD2|NPAS2|NR3C1|NUP62|PACRG|PDXP|PPEF2|PPID|PPP5C|PRKN|PTGES3|RNF207|RPS3|SACS|SNCA|SPN|ST13|STIP1|STUB1|TELO2|TFAM|TOMM34|TPR|TSC1|TSC2|USP19|ZFP36|                                                                                                                                                                                                                                                                                                                                                                                   |
| response to topologically incorrect protein | <GO:0035966> |       162 |      162 |       14 |    0 |               0 |        0 |                 0 |              0.911 | NA      | ACADVL|ADD1|AMFR|ARFGAP1|ASNA1|ASNS|ATF3|ATF4|ATF6|ATF6B|ATP6V0D1|ATXN3|BAG3|BAG6|BAK1|BAX|BHLHA15|CALR|CCL2|CCND1|CDK5RAP3|CHAC1|CLU|COMP|CREB3|CREB3L1|CREB3L2|CREB3L3|CREB3L4|CREBRF|CTDSP2|CTH|CUL3|CUL7|CXCL8|CXXC1|DAB2IP|DAXX|DCTN1|DDIT3|DDX11|DERL1|DERL2|DERL3|DNAJA1|DNAJB1|DNAJB11|DNAJB12|DNAJB2|DNAJB4|DNAJB5|DNAJB9|DNAJC3|DNAJC4|EDEM1|EDEM2|EDEM3|EIF2AK2|EIF2AK3|EIF2S1|EP300|ERN1|ERO1A|ERP44|EXTL1|EXTL2|EXTL3|F12|FAF2|FBXO6|FGF21|FICD|FKBP14|GFPT1|GOSR2|GSK3A|HDAC6|HDGF|HERPUD1|HERPUD2|HSF1|HSP90AA1|HSP90AB1|HSP90B1|HSPA1A|HSPA4|HSPA4L|HSPA5|HSPA8|HSPB1|HSPB2|HSPB3|HSPB7|HSPB8|HSPD1|HSPE1|HSPH1|HYOU1|IGFBP1|JKAMP|KDELR3|KLHDC3|KLHL15|LMNA|MANF|MBTPS1|MBTPS2|MFN2|MMP24-AS1-EDEM2|MYDGF|NFE2L2|OPTN|PACRG|PARP16|PDIA5|PDIA6|PLA2G4B|PPP2R5B|PREB|PRKN|PTPN1|RHBDD1|RNF126|RNF5|SDF2L1|SEC31A|SELENOS|SERP1|SERPINH1|SHC1|SRPRA|SRPRB|SSR1|STC2|STT3B|STUB1|SULT1A3|SYVN1|TATDN2|TBL2|THBS1|THBS4|TLN1|TM7SF3|TMBIM6|TMEM129|TOR1A|TOR1B|TPP1|TSPYL2|UBE2J2|UBE2W|UBXN4|UFD1|VAPB|VCP|WFS1|WIPI1|XBP1|YIF1A|YOD1|ZBTB17|                   |

We can see <GO:0051082> is the top scoring hit as expected.
