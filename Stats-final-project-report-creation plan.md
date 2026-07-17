



## Final methodological decision

**Use R and RStudio.** That is the correct choice for this project because:

- the module already uses R and R Markdown;
- the project is fundamentally psychometric and inferential;
- R provides established packages for reliability analysis, polychoric correlations, parallel analysis, exploratory factor analysis, effect sizes, regression diagnostics, and reproducible reporting;
- submitting both the rendered PDF and the underlying `.Rmd` files will demonstrate reproducibility.

The `psych` package directly supports Cronbach’s alpha, McDonald’s omega, polychoric correlations, parallel analysis, and factor analysis. Its `fa.parallel()` function compares observed eigenvalues with eigenvalues from random data to help determine factor count. citeturn570075search0turn570075search1

Your finalized instrument currently contains **27 adapted MSLQ items across seven subscales**, plus demographic/contextual variables, eligibility checks, an attention check, and an open-ended question. fileciteturn6file0 The original MSLQ was designed to assess higher-education students’ motivational orientations and use of learning strategies, so it is conceptually appropriate for your study. citeturn570075search4turn570075search20

---

# 1. Understand the two reports correctly

The two reports answer different questions.

| Report | Main question |
|---|---|
| **Validation Report** | Does the adapted questionnaire measure the intended constructs reliably and plausibly? |
| **Data Analysis Report** | What do the respondents’ scores show, and which student/contextual factors are associated with self-regulated learning? |

Do not copy the same results into both reports.

The **Validation Report** focuses on the instrument.

The **Data Analysis Report** focuses on the respondents and the research hypotheses.

---

# 2. Critical corrections before analysis

Before any R analysis, complete these tasks.

## 2.1 Freeze the final questionnaire

The questionnaire document, Google Form, response spreadsheet, and report must match exactly.

Update your finalized questionnaire document to reflect the actual form:

- grouped programme-area options;
- open-ended challenge question marked as required;
- email collection turned off;
- final wording of all questions;
- final attention-check wording;
- exact 27 MSLQ items.

Do not analyze responses collected under materially different questionnaire versions as though they came from one unchanged instrument.

## 2.2 Preserve an untouched raw dataset

Export the final Google Sheet as:

> `MSLQ_Raw_Responses.xlsx`

Never manually clean or edit this file.

Make a copy for analysis:

> `MSLQ_Working_Copy.xlsx`

However, the better method is to keep the raw Excel file untouched and perform all cleaning through R.

## 2.3 Remove test submissions

Remove known test responses before analysis.

Document:

- how many test responses were removed;
- why they were removed;
- the final number of genuine responses.

## 2.4 Do not claim sampling was representative unless it truly was

You probably used targeted distribution through postgraduate groups and personal academic networks. That is most likely:

> **Purposive and convenience sampling, supplemented by snowball sharing.**

Do not call it probability sampling or a nationally representative sample unless respondents were randomly selected from a complete sampling frame.

## 2.5 Do not fabricate a pilot study

Use the correct route:

### Route A — A genuine separate pilot was conducted

Report:

- pilot sample size;
- pilot recruitment;
- respondent feedback;
- preliminary reliability;
- questionnaire modifications;
- whether pilot responses were excluded from the main analysis.

### Route B — No separate pilot was conducted

State honestly:

> A separate independent pilot sample was not available. The adapted instrument was evaluated psychometrically using the final cross-sectional study sample. Consequently, test-retest stability and independent confirmation of the factor structure were not assessed.

This is a limitation, but it is far better than inventing a pilot.

---

# 3. Final conceptual model and scoring strategy

Your 27 items belong to seven subscales:

| Code | Construct | Items |
|---|---|---:|
| SE | Self-efficacy | 4 |
| TV | Task value | 4 |
| IM | Intrinsic motivation | 3 |
| MSR | Metacognitive self-regulation | 5 |
| ER | Effort regulation | 4 |
| TM | Time/study management | 4 |
| HS | Help-seeking/peer learning | 3 |

## 3.1 Recommended scores

Calculate the following:

### Seven primary subscale scores

Each score is the mean of its items:

- Self-efficacy
- Task value
- Intrinsic motivation
- Metacognitive self-regulation
- Effort regulation
- Time/study management
- Help-seeking/peer learning

All scores remain on the interpretable **1–5 scale**.

### Motivation composite

Calculate:

> Mean of SE1–SE4, TV1–TV4, and IM1–IM3

This combines the 11 motivational items.

### Self-regulated learning strategy composite

Calculate:

> Mean of MSR1–MSR5, ER1–ER4, TM1–TM4, and HS1–HS3

This combines the 16 learning-strategy items.

### Overall 27-item score

You may calculate this for exploratory description, but **do not use it as your principal outcome unless the factor analysis supports an overarching general factor**.

## 3.2 Important regression rule

Do not run a model such as:

> Overall 27-item score predicted by self-efficacy, task value, and effort regulation

That is circular because those predictors are already included inside the overall score.

Use:

> **SRL strategy composite** as the outcome  
> **Self-efficacy, task value, and intrinsic motivation** as predictors

This is conceptually cleaner.

## 3.3 Reverse scoring

Your finalized adapted instrument contains positively worded items only. Therefore, based on the submitted item set:

> **No item requires reverse scoring.**

Mention this explicitly in the scoring method.

Also acknowledge as a limitation that using only positively phrased items may increase acquiescence or agreement-response bias.

---

# 4. Proposed research questions and hypotheses

Use a controlled number of research questions. Do not test every demographic variable merely because it is available.

## Validation questions

**VRQ1:** Do the adapted MSLQ subscales demonstrate satisfactory internal consistency?

**VRQ2:** Does the factor structure of the adapted items provide reasonable support for the proposed seven-dimensional structure?

**VRQ3:** Are individual items sufficiently related to their intended subscales without excessive redundancy or cross-loading?

## Data-analysis questions

**RQ1:** What are the levels of motivation and self-regulated learning among postgraduate computing students?

**RQ2:** Are motivation dimensions significantly associated with students’ self-regulated learning strategies?

**RQ3:** Is AI-tool usage associated with self-regulated learning after controlling for academic and personal factors?

**RQ4:** Do employed and non-employed postgraduate students differ in self-regulated learning?

## Primary hypotheses

### H1 — Motivation and SRL

- **H₀:** Self-efficacy, task value, and intrinsic motivation are not significantly associated with the self-regulated learning strategy score.
- **H₁:** At least one motivational dimension is significantly associated with the self-regulated learning strategy score.

### H2 — AI-tool usage

- **H₀:** AI-tool usage frequency is not associated with self-regulated learning after adjusting for other variables.
- **H₁:** AI-tool usage frequency is associated with self-regulated learning after adjusting for other variables.

Use “associated with,” not “causes” or “influences,” because this is cross-sectional observational data.

### H3 — Employment status

- **H₀:** The mean self-regulated learning strategy score is equal for employed and non-employed students.
- **H₁:** The mean self-regulated learning strategy score differs between employed and non-employed students.

Programme-area, delivery-mode, and degree-level comparisons can be treated as **secondary or exploratory analyses**.

---

# 5. Complete Validation Report plan

## Recommended title

**Validation Report: Adapted Motivated Strategies for Learning Questionnaire for Postgraduate Computing Students**

## Recommended structure

### Cover page

Include:

- university and department;
- module name and code;
- report title;
- study title;
- group members;
- submission date.

### Executive summary

Approximately 200–300 words covering:

- purpose of adaptation;
- sample size;
- validation methods;
- principal reliability findings;
- factor-analysis findings;
- overall conclusion;
- major limitation.

Do not write the final summary until all analysis is completed.

---

## Section 1 — Introduction

Explain:

- what the MSLQ measures;
- why adaptation was required;
- why postgraduate computing students constitute a distinctive context;
- why motivation and self-regulation matter for technical postgraduate learning;
- validation objectives.

Do not repeat the full literature review intended for the conference paper.

---

## Section 2 — Instrument adaptation

Include:

### 2.1 Original instrument

Briefly describe:

- original MSLQ;
- original authors;
- motivational and learning-strategy components;
- higher-education context.

### 2.2 Contextual adaptation

Explain changes such as:

- general “course” language changed to postgraduate computing;
- references to programming, statistics, machine learning, research, projects, and AI;
- reduced selection of relevant subscales;
- five-point response scale used in the adapted version.

### 2.3 Original-to-adapted item table

Include a table with:

| Original item/concept | Adapted item | Construct | Reason for change |
|---|---|---|---|

This can be placed in an appendix if it is long.

### 2.4 Content and face review

The strongest feasible approach is to ask **3–5 reviewers** to assess each item for:

- relevance;
- clarity;
- contextual appropriateness.

Ideal reviewer roles:

- statistics/psychometrics lecturer;
- postgraduate computing lecturer;
- Data Science/AI academic;
- educational-research specialist;
- senior postgraduate student for readability.

Use a 4-point rating:

1. Not relevant/clear  
2. Needs major revision  
3. Relevant/clear with minor revision  
4. Highly relevant/clear  

Calculate:

- Item Content Validity Index, I-CVI;
- Scale Content Validity Index average, S-CVI/Ave.

CVI commonly summarizes the proportion of experts assigning acceptable relevance ratings to each item and the average across the scale. citeturn190168search3turn190168search11

If expert ratings were not collected, report only a qualitative review. Do not invent numerical CVI results.

---

## Section 3 — Validation methodology

### 3.1 Study design

State:

> Cross-sectional questionnaire adaptation and psychometric evaluation study.

### 3.2 Participants and recruitment

Report:

- inclusion criteria;
- exclusion criteria;
- recruitment channels;
- sampling technique;
- collection dates;
- raw number of submissions;
- final valid sample.

### 3.3 Ethical procedures

Report:

- informed consent;
- anonymous responses;
- no email collection;
- voluntary participation;
- data confidentiality;
- aggregated reporting.

### 3.4 Data-quality criteria

Predefine exclusions:

- no consent;
- not a current postgraduate student;
- not in a computing-related programme;
- attention-check failure;
- known test response;
- exact duplicate submission;
- materially incomplete response.

Do not automatically delete someone only because all their Likert answers are similar. Flag straight-lining and review it together with the attention check and other evidence.

### 3.5 Scoring

Describe:

- Likert coding from 1 to 5;
- no reverse scoring;
- mean-based subscale scores;
- 80% completion rule if any item is missing;
- motivation and SRL strategy composites.

### 3.6 Reliability analysis

Calculate separately for each subscale:

- Cronbach’s alpha;
- standardized alpha;
- McDonald’s omega, where estimable;
- corrected item-total correlation;
- alpha if item deleted.

Cronbach’s alpha is not sufficient by itself. Omega provides complementary reliability information, while item diagnostics show whether a particular question is weak or redundant. The `psych` package supports both alpha and omega. citeturn570075search0turn570075search15

### 3.7 Construct validity

Because the responses are five-category Likert data:

1. calculate a polychoric correlation matrix;
2. calculate KMO;
3. perform Bartlett’s test;
4. conduct parallel analysis;
5. conduct exploratory factor analysis using:
   - minimum residual extraction;
   - oblimin rotation because the constructs are theoretically related.

Parallel analysis should inform the factor count rather than automatically forcing seven factors. citeturn570075search1turn570075search28

Also fit the theory-driven seven-factor EFA and compare its interpretability with the parallel-analysis recommendation.

### 3.8 CFA decision

Do not perform EFA and CFA on the same sample and present this as independent confirmation.

With approximately 200 responses and 27 items, the safest core analysis is:

> **EFA as the main construct-validity analysis.**

A CFA using ordinal indicators and `WLSMV` can be included only as supplementary analysis, preferably with a larger or independent sample. In `lavaan`, specifying items as ordered automatically invokes a WLSMV-type estimator. citeturn570075search2

### 3.9 Test-retest reliability

Only report this if the same participants answered the questionnaire twice and could be matched anonymously.

If this was not collected, write:

> Test-retest reliability was not assessed because a second matched measurement wave was unavailable.

---

## Section 4 — Validation results

Present results in this order.

### Table V1 — Response-quality flow

| Stage | n |
|---|---:|
| Total submissions | |
| Known tests removed | |
| No consent | |
| Ineligible respondents | |
| Attention-check failures | |
| Exact duplicates | |
| Final validation sample | |

### Table V2 — Item descriptive statistics

Include:

- item code;
- mean;
- standard deviation;
- median;
- minimum/maximum;
- percentage choosing 1;
- percentage choosing 5.

### Table V3 — Reliability by subscale

| Subscale | Items | Alpha | Omega | Interpretation |
|---|---:|---:|---:|---|

### Table V4 — Item diagnostics

| Item | Corrected item-total correlation | Alpha if deleted | Decision |
|---|---:|---:|---|

Do not remove items merely because alpha increases slightly. Consider:

- theoretical importance;
- item wording;
- factor loading;
- cross-loading;
- redundancy;
- expert comments.

### Table V5 — Factorability

Include:

- overall KMO;
- item-level KMO range;
- Bartlett chi-square;
- degrees of freedom;
- p-value.

### Figure V1 — Parallel-analysis plot

Show observed and simulated eigenvalues.

### Table V6 — Rotated factor loadings

Show loadings for each item across retained factors.

Use the following as **diagnostic working rules**, not automatic laws:

- primary loading around .40 or higher is desirable;
- large cross-loadings require review;
- a factor should ideally have at least three meaningful items;
- strongly correlated factors may indicate insufficient distinction.

### Figure V2 — Factor-loading heatmap or factor diagram

Optional but visually effective.

---

## Section 5 — Validation discussion

Interpret:

- which subscales were most reliable;
- whether any scale was weak;
- whether the seven-factor expectation was supported;
- whether any items cross-loaded;
- whether adaptations may have altered item meaning;
- whether broad computing disciplines responded similarly;
- whether the instrument is ready for main interpretation.

Avoid writing:

> The questionnaire is fully validated.

Use:

> The adapted questionnaire demonstrated preliminary/acceptable psychometric evidence in the present sample.

Validation is evidence accumulated across studies, not a permanent label obtained from one dataset.

---

## Section 6 — Limitations and conclusion

Mandatory limitations:

- convenience/purposive sampling;
- cross-sectional design;
- self-report bias;
- all positively worded items;
- broad mixture of computing programmes;
- no test-retest data, if unavailable;
- EFA and inference from the same sample;
- lack of external criterion measure;
- no independent CFA sample.

---

# 6. Complete Data Analysis Report plan

## Recommended title

**Data Analysis Report: Self-Regulated Learning among Postgraduate Computing Students**

## Section 1 — Research objectives and hypotheses

State:

- research objectives;
- primary hypotheses;
- secondary/exploratory analyses;
- significance level, normally α = .05;
- use of 95% confidence intervals.

Distinguish analyses planned before seeing outcomes from exploratory analyses.

---

## Section 2 — Data and methods

### 2.1 Final analytical sample

Report the final valid sample after cleaning.

### 2.2 Variables

#### Outcome

Primary outcome:

> SRL strategy composite: MSR + ER + TM + HS items

#### Main predictors

- self-efficacy;
- task value;
- intrinsic motivation;
- AI-tool usage;
- weekly study hours;
- mathematics/statistics confidence;
- employment;
- study mode.

#### Secondary variables

- programme group;
- delivery mode;
- degree level;
- programming experience;
- academic stage;
- age group;
- gender.

### 2.3 Statistical methods

Use:

- counts and percentages for categorical variables;
- means, standard deviations, medians, IQRs, and 95% CIs for scores;
- correlation analysis;
- Welch independent-samples t-test for two-group comparisons;
- ANOVA or Welch ANOVA for multiple groups;
- post-hoc tests only after a significant omnibus test;
- multiple linear regression;
- effect sizes;
- adjusted p-values for numerous exploratory tests.

Effect-size reporting is essential because a p-value does not communicate the magnitude of a difference. The `effectsize` package supports standardized mean differences and ANOVA effect sizes. citeturn570075search3turn570075search18

---

## Section 3 — Results

### Table D1 — Sample profile

Include:

- age;
- gender;
- degree level;
- programme category;
- study mode;
- delivery mode;
- employment;
- academic stage;
- weekly study hours;
- AI-tool usage;
- programming experience;
- mathematics/statistics confidence.

### Table D2 — Subscale descriptives

| Score | n | Mean | SD | Median | IQR | 95% CI |
|---|---:|---:|---:|---:|---:|---|

### Figure D1 — Mean scores with 95% confidence intervals

Show the seven subscale means.

### Figure D2 — Subscale correlation matrix

Show relationships among:

- SE;
- TV;
- IM;
- MSR;
- ER;
- TM;
- HS.

### Table D3 — Primary group comparison

For employment status:

- group sizes;
- group means;
- mean difference;
- 95% CI;
- Welch t;
- degrees of freedom;
- p-value;
- Hedges’ g.

### Table D4 — Programme or study-mode comparison

Only include this if groups have usable sample sizes.

For programme area:

- omnibus ANOVA/Welch ANOVA;
- effect size;
- post-hoc comparisons if significant.

### Table D5 — Multiple regression

Outcome:

> SRL strategy composite

Predictors:

- self-efficacy;
- task value;
- intrinsic motivation;
- AI-use frequency;
- weekly study-hours category;
- mathematics/statistics confidence;
- study mode;
- employment status.

Report:

- unstandardized coefficient;
- standard error;
- 95% CI;
- standardized coefficient;
- p-value;
- model R²;
- adjusted R²;
- overall F-test;
- VIF;
- robust HC3 standard errors if heteroscedasticity exists.

### Figure D3 — Regression coefficient plot

Optional but highly effective.

### Table D6 — Qualitative challenge themes

Because your open-ended question is required, conduct a small supporting thematic content analysis.

Two group members should:

1. read all responses;
2. independently assign short theme labels;
3. compare and resolve differences;
4. group responses into final themes;
5. report counts and percentages.

Possible themes may include:

- time management;
- work-study balance;
- technical difficulty;
- research workload;
- motivation;
- resource limitations;
- group-work issues;
- no major challenge.

Do not create themes in advance and force responses into them. Let the data determine the final themes.

---

## Section 4 — Discussion

Discuss:

- major level of each construct;
- strongest and weakest learning dimensions;
- important predictors;
- practical meaning of AI-use findings;
- whether employed students differ;
- implications for programme design;
- comparison with previous literature;
- effect sizes, not merely significance.

Do not claim that AI-tool usage caused stronger or weaker SRL.

---

## Section 5 — Limitations and conclusion

Include the sampling, cross-sectional, self-report, and validation limitations.

End with a conclusion directly connected to the hypotheses.

---

# 7. RStudio project structure

Create one RStudio Project:

```text
MSLQ_Project/
│
├── MSLQ_Project.Rproj
├── data_raw/
│   └── MSLQ_Raw_Responses.xlsx
├── data_processed/
│   ├── MSLQ_Analysis_Data.csv
│   ├── MSLQ_Excluded_Responses.csv
│   └── Codebook.xlsx
├── R/
│   ├── 00_setup.R
│   ├── 01_clean_and_score.R
│   ├── 02_validation_analysis.R
│   └── 03_data_analysis.R
├── reports/
│   ├── Validation_Report.Rmd
│   └── Data_Analysis_Report.Rmd
├── outputs/
│   ├── tables/
│   └── figures/
└── references/
    └── references.bib
```

Use an RStudio Project rather than `setwd()` paths.

For reproducibility, initialize `renv`; it records the project package environment in a lockfile so another group member can restore the same package versions. citeturn759960search1turn759960search5

---

# 8. Setup script

Save as:

> `R/00_setup.R`

```r
# ============================================================
# CS5651 MSLQ PROJECT: PACKAGE SETUP
# ============================================================

required_packages <- c(
  "tidyverse",
  "readxl",
  "janitor",
  "psych",
  "GPArotation",
  "effectsize",
  "rstatix",
  "car",
  "lmtest",
  "sandwich",
  "broom",
  "parameters",
  "knitr",
  "kableExtra",
  "writexl",
  "renv"
)

missing_packages <- setdiff(
  required_packages,
  rownames(installed.packages())
)

if (length(missing_packages) > 0) {
  install.packages(missing_packages)
}

invisible(
  lapply(required_packages, library, character.only = TRUE)
)

set.seed(5651)

dir.create("data_processed", showWarnings = FALSE)
dir.create("outputs", showWarnings = FALSE)
dir.create("outputs/tables", showWarnings = FALSE)
dir.create("outputs/figures", showWarnings = FALSE)

options(
  dplyr.summarise.inform = FALSE,
  scipen = 999
)

# Run only once when setting up the project:
# renv::init()

# Run after the analysis is working:
# renv::snapshot()
```

---

# 9. Complete cleaning and scoring script

Save as:

> `R/01_clean_and_score.R`

This code searches for the exact item statements inside Google Forms column headings, so it is more robust than manually using Excel column positions.

```r
source("R/00_setup.R")

raw_path <- "data_raw/MSLQ_Raw_Responses.xlsx"

if (!file.exists(raw_path)) {
  stop("Raw response file not found: ", raw_path)
}

raw <- readxl::read_excel(
  raw_path,
  .name_repair = "unique"
)

# ------------------------------------------------------------
# Helper: find exactly one column containing a question pattern
# ------------------------------------------------------------

find_question_column <- function(column_names, pattern) {
  hits <- column_names[
    stringr::str_detect(
      stringr::str_to_lower(column_names),
      stringr::fixed(stringr::str_to_lower(pattern))
    )
  ]

  if (length(hits) != 1) {
    stop(
      "Expected exactly one column for pattern:\n",
      pattern,
      "\nFound: ",
      paste(hits, collapse = " | ")
    )
  }

  hits
}

rename_using_patterns <- function(data, pattern_map) {
  old_names <- vapply(
    pattern_map,
    function(pattern) find_question_column(names(data), pattern),
    character(1)
  )

  names(data)[match(old_names, names(data))] <- names(pattern_map)
  data
}

# ------------------------------------------------------------
# Contextual and screening questions
# ------------------------------------------------------------

question_patterns <- c(
  consent = "Do you agree to participate",
  postgraduate_status = "Are you currently enrolled in a postgraduate",
  computing_related = "Is your postgraduate programme related to computing",
  age_group = "What is your age group",
  gender = "What is your gender",
  degree_level = "What is your current postgraduate degree level",
  programme_area = "What is your main programme area",
  study_mode = "What is your current study mode",
  delivery_mode = "What is the main delivery mode",
  employment_status = "What is your current employment status",
  academic_stage = "What is your current academic stage",
  weekly_study_hours = "how many hours per week do you spend",
  ai_use = "How often do you use AI tools",
  programming_experience = "How would you describe your prior programming",
  math_statistics_confidence = "How confident are you in mathematics",
  attention_check = "To confirm that you are reading",
  main_challenge = "What is the main challenge you face"
)

dat <- rename_using_patterns(raw, question_patterns)

# ------------------------------------------------------------
# The 27 adapted MSLQ item statements
# ------------------------------------------------------------

item_text <- c(
  SE1 = "I am confident that I can understand complex concepts in Data Science, AI, or related computing modules.",
  SE2 = "I believe I can successfully complete assignments, projects, or practical tasks in my postgraduate computing programme.",
  SE3 = "I am confident that I can learn difficult programming, statistics, or machine learning topics if I put in enough effort.",
  SE4 = "Compared with other students in my programme, I believe I can perform well in academic tasks.",

  TV1 = "I believe that the topics taught in my programme are useful for my academic and professional development.",
  TV2 = "Learning Data Science, AI, or computing-related concepts is important for my future goals.",
  TV3 = "The assignments and projects in my programme help me develop valuable practical skills.",
  TV4 = "I find the content of my postgraduate computing programme meaningful and worth learning.",

  IM1 = "I study computing-related topics because I am genuinely interested in learning them.",
  IM2 = "I enjoy learning new methods, tools, or technologies related to Data Science and AI.",
  IM3 = "I prefer learning tasks that challenge me to think deeply and solve problems.",

  MSR1 = "Before studying, I usually plan what topics or tasks I need to complete.",
  MSR2 = "While studying, I check whether I really understand the concepts and methods.",
  MSR3 = "If I do not understand a topic, I change my learning strategy and try another approach.",
  MSR4 = "I connect new topics with previous knowledge from computing, statistics, or programming.",
  MSR5 = "I review my mistakes in assignments, coding tasks, or projects to improve my learning.",

  ER1 = "Even when coursework becomes difficult, I continue working until I make progress.",
  ER2 = "I keep trying when I face challenges in coding, statistics, machine learning, or research tasks.",
  ER3 = "I do not easily give up on difficult academic tasks in my postgraduate programme.",
  ER4 = "I complete important learning tasks even when they require extra time and effort.",

  TM1 = "I manage my study time effectively to complete lectures, assignments, and projects.",
  TM2 = "I set aside regular time for independent learning outside scheduled classes.",
  TM3 = "I organize my study environment to reduce distractions when learning.",
  TM4 = "I balance academic work with other responsibilities such as employment, family, or personal commitments.",

  HS1 = "When I do not understand a topic, I seek help from lecturers, classmates, or reliable learning resources.",
  HS2 = "I discuss difficult computing-related concepts or assignments with peers when needed.",
  HS3 = "I use appropriate academic or digital resources to clarify difficult learning problems."
)

dat <- rename_using_patterns(dat, item_text)

item_codes <- names(item_text)

# ------------------------------------------------------------
# Likert conversion
# Handles both "4 Agree" and "Agree"
# ------------------------------------------------------------

likert_to_numeric <- function(x) {
  x <- stringr::str_squish(as.character(x))

  numeric_value <- suppressWarnings(
    readr::parse_number(x)
  )

  word_map <- c(
    "strongly disagree" = 1,
    "disagree" = 2,
    "neutral" = 3,
    "agree" = 4,
    "strongly agree" = 5
  )

  missing_number <- is.na(numeric_value)

  numeric_value[missing_number] <- unname(
    word_map[stringr::str_to_lower(x[missing_number])]
  )

  as.numeric(numeric_value)
}

dat <- dat %>%
  mutate(
    across(all_of(item_codes), likert_to_numeric),
    math_statistics_confidence =
      suppressWarnings(readr::parse_number(
        as.character(math_statistics_confidence)
      ))
  )

# ------------------------------------------------------------
# Data-quality flags
# ------------------------------------------------------------

clean_attention <- stringr::str_to_lower(
  stringr::str_squish(
    stringr::str_remove(
      as.character(dat$attention_check),
      "^[1-5]\\s*"
    )
  )
)

dat <- dat %>%
  mutate(
    consent_valid =
      stringr::str_detect(
        stringr::str_to_lower(as.character(consent)),
        "^i agree"
      ),

    postgraduate_valid =
      stringr::str_to_lower(
        stringr::str_squish(as.character(postgraduate_status))
      ) == "yes",

    computing_valid =
      stringr::str_to_lower(
        stringr::str_squish(as.character(computing_related))
      ) == "yes",

    attention_valid = clean_attention == "agree"
  )

# Exact duplicate signature excluding timestamp
signature_columns <- c(
  "age_group",
  "gender",
  "degree_level",
  "programme_area",
  "study_mode",
  "delivery_mode",
  "employment_status",
  "academic_stage",
  "weekly_study_hours",
  "ai_use",
  "programming_experience",
  "math_statistics_confidence",
  item_codes,
  "main_challenge"
)

duplicate_signature <- do.call(
  paste,
  c(dat[signature_columns], sep = "\r")
)

dat$exact_duplicate <- duplicated(duplicate_signature)

# Straight-lining is flagged, not automatically excluded
dat$within_person_item_sd <- apply(
  dat[item_codes],
  1,
  sd,
  na.rm = TRUE
)

dat$straightline_flag <-
  !is.na(dat$within_person_item_sd) &
  dat$within_person_item_sd == 0

dat <- dat %>%
  mutate(
    exclusion_reason = case_when(
      !consent_valid ~ "Consent not provided",
      !postgraduate_valid ~ "Not currently postgraduate",
      !computing_valid ~ "Not computing-related",
      !attention_valid ~ "Attention check failed",
      exact_duplicate ~ "Exact duplicate",
      TRUE ~ NA_character_
    )
  )

excluded_dat <- dat %>%
  filter(!is.na(exclusion_reason))

analysis_dat <- dat %>%
  filter(is.na(exclusion_reason))

# ------------------------------------------------------------
# Subscale scoring
# ------------------------------------------------------------

subscale_items <- list(
  self_efficacy = c("SE1", "SE2", "SE3", "SE4"),
  task_value = c("TV1", "TV2", "TV3", "TV4"),
  intrinsic_motivation = c("IM1", "IM2", "IM3"),
  metacognitive_regulation =
    c("MSR1", "MSR2", "MSR3", "MSR4", "MSR5"),
  effort_regulation = c("ER1", "ER2", "ER3", "ER4"),
  time_management = c("TM1", "TM2", "TM3", "TM4"),
  help_seeking = c("HS1", "HS2", "HS3")
)

score_mean <- function(data, variables, minimum_proportion = 0.80) {
  matrix_data <- as.matrix(data[variables])
  valid_count <- rowSums(!is.na(matrix_data))
  required_count <- ceiling(length(variables) * minimum_proportion)

  score <- rowMeans(matrix_data, na.rm = TRUE)
  score[valid_count < required_count] <- NA_real_

  score
}

for (scale_name in names(subscale_items)) {
  analysis_dat[[scale_name]] <- score_mean(
    analysis_dat,
    subscale_items[[scale_name]]
  )
}

motivation_items <- c(
  subscale_items$self_efficacy,
  subscale_items$task_value,
  subscale_items$intrinsic_motivation
)

srl_strategy_items <- c(
  subscale_items$metacognitive_regulation,
  subscale_items$effort_regulation,
  subscale_items$time_management,
  subscale_items$help_seeking
)

analysis_dat$motivation_composite <- score_mean(
  analysis_dat,
  motivation_items
)

analysis_dat$srl_strategy_composite <- score_mean(
  analysis_dat,
  srl_strategy_items
)

analysis_dat$overall_adapted_score <- score_mean(
  analysis_dat,
  item_codes
)

# ------------------------------------------------------------
# Save outputs
# ------------------------------------------------------------

readr::write_csv(
  analysis_dat,
  "data_processed/MSLQ_Analysis_Data.csv"
)

readr::write_csv(
  excluded_dat,
  "data_processed/MSLQ_Excluded_Responses.csv"
)

quality_flow <- tibble(
  stage = c(
    "Raw submissions",
    "Valid consent",
    "Eligible postgraduate",
    "Computing-related",
    "Passed attention check",
    "After duplicate removal",
    "Final analytical sample"
  ),
  n = c(
    nrow(dat),
    sum(dat$consent_valid, na.rm = TRUE),
    sum(dat$consent_valid &
          dat$postgraduate_valid, na.rm = TRUE),
    sum(dat$consent_valid &
          dat$postgraduate_valid &
          dat$computing_valid, na.rm = TRUE),
    sum(dat$consent_valid &
          dat$postgraduate_valid &
          dat$computing_valid &
          dat$attention_valid, na.rm = TRUE),
    sum(is.na(dat$exclusion_reason), na.rm = TRUE),
    nrow(analysis_dat)
  )
)

readr::write_csv(
  quality_flow,
  "outputs/tables/data_quality_flow.csv"
)

print(quality_flow)
```

---

# 10. Validation analysis script

Save as:

> `R/02_validation_analysis.R`

```r
source("R/00_setup.R")
source("R/01_clean_and_score.R")

item_data <- analysis_dat %>%
  select(all_of(item_codes))

# ------------------------------------------------------------
# Item descriptive statistics
# ------------------------------------------------------------

item_statistics <- item_data %>%
  pivot_longer(
    cols = everything(),
    names_to = "item",
    values_to = "response"
  ) %>%
  group_by(item) %>%
  summarise(
    n = sum(!is.na(response)),
    mean = mean(response, na.rm = TRUE),
    sd = sd(response, na.rm = TRUE),
    median = median(response, na.rm = TRUE),
    minimum = min(response, na.rm = TRUE),
    maximum = max(response, na.rm = TRUE),
    floor_percent = mean(response == 1, na.rm = TRUE) * 100,
    ceiling_percent = mean(response == 5, na.rm = TRUE) * 100
  )

write_csv(
  item_statistics,
  "outputs/tables/item_statistics.csv"
)

# ------------------------------------------------------------
# Reliability analysis
# ------------------------------------------------------------

reliability_one_scale <- function(scale_name, variables) {
  x <- analysis_dat %>%
    select(all_of(variables))

  alpha_result <- psych::alpha(
    x,
    check.keys = FALSE,
    warnings = FALSE
  )

  omega_value <- tryCatch(
    {
      omega_result <- psych::omega(
        x,
        nfactors = 1,
        plot = FALSE
      )
      omega_result$omega.tot
    },
    error = function(e) NA_real_
  )

  tibble(
    subscale = scale_name,
    number_of_items = length(variables),
    cronbach_alpha = alpha_result$total$raw_alpha,
    standardized_alpha = alpha_result$total$std.alpha,
    mcdonald_omega = omega_value
  )
}

reliability_table <- purrr::imap_dfr(
  subscale_items,
  reliability_one_scale
)

write_csv(
  reliability_table,
  "outputs/tables/reliability_by_subscale.csv"
)

# ------------------------------------------------------------
# Item-total diagnostics
# ------------------------------------------------------------

item_diagnostics <- purrr::imap_dfr(
  subscale_items,
  function(variables, scale_name) {
    x <- analysis_dat %>%
      select(all_of(variables))

    result <- psych::alpha(
      x,
      check.keys = FALSE,
      warnings = FALSE
    )

    tibble(
      subscale = scale_name,
      item = rownames(result$item.stats),
      corrected_item_total = result$item.stats$r.drop,
      alpha_if_deleted = result$alpha.drop$raw_alpha
    )
  }
)

write_csv(
  item_diagnostics,
  "outputs/tables/item_diagnostics.csv"
)

# ------------------------------------------------------------
# Polychoric correlation matrix
# ------------------------------------------------------------

poly_result <- psych::polychoric(item_data)
poly_matrix <- poly_result$rho

write.csv(
  poly_matrix,
  "outputs/tables/polychoric_correlation_matrix.csv"
)

# ------------------------------------------------------------
# KMO and Bartlett tests
# ------------------------------------------------------------

kmo_result <- psych::KMO(poly_matrix)

bartlett_result <- psych::cortest.bartlett(
  poly_matrix,
  n = nrow(item_data)
)

factorability_table <- tibble(
  statistic = c(
    "Overall KMO",
    "Bartlett chi-square",
    "Bartlett degrees of freedom",
    "Bartlett p-value"
  ),
  value = c(
    kmo_result$MSA,
    bartlett_result$chisq,
    bartlett_result$df,
    bartlett_result$p.value
  )
)

write_csv(
  factorability_table,
  "outputs/tables/factorability_results.csv"
)

# ------------------------------------------------------------
# Parallel analysis
# ------------------------------------------------------------

png(
  "outputs/figures/parallel_analysis.png",
  width = 1800,
  height = 1200,
  res = 200
)

parallel_result <- psych::fa.parallel(
  poly_matrix,
  n.obs = nrow(item_data),
  fa = "fa",
  fm = "minres",
  main = "Parallel Analysis of Adapted MSLQ Items"
)

dev.off()

recommended_factors <- parallel_result$nfact

message(
  "Parallel analysis recommended ",
  recommended_factors,
  " factor(s)."
)

# ------------------------------------------------------------
# Theory-driven seven-factor EFA
# ------------------------------------------------------------

efa_seven <- psych::fa(
  r = poly_matrix,
  nfactors = 7,
  n.obs = nrow(item_data),
  fm = "minres",
  rotate = "oblimin"
)

capture.output(
  print(efa_seven$loadings, cutoff = 0.30, sort = TRUE),
  file = "outputs/tables/efa_seven_factor_loadings.txt"
)

loading_table <- as.data.frame(
  unclass(efa_seven$loadings)
)

loading_table$item <- rownames(loading_table)

loading_table <- loading_table %>%
  relocate(item)

write_csv(
  loading_table,
  "outputs/tables/efa_seven_factor_loadings.csv"
)

# ------------------------------------------------------------
# Empirically recommended EFA
# ------------------------------------------------------------

efa_parallel <- psych::fa(
  r = poly_matrix,
  nfactors = recommended_factors,
  n.obs = nrow(item_data),
  fm = "minres",
  rotate = "oblimin"
)

capture.output(
  print(efa_parallel$loadings, cutoff = 0.30, sort = TRUE),
  file = "outputs/tables/efa_parallel_solution.txt"
)

# ------------------------------------------------------------
# Factor-loading heatmap for seven-factor solution
# ------------------------------------------------------------

loading_long <- loading_table %>%
  pivot_longer(
    cols = -item,
    names_to = "factor",
    values_to = "loading"
  )

loading_plot <- ggplot(
  loading_long,
  aes(x = factor, y = item, fill = loading)
) +
  geom_tile() +
  geom_text(
    aes(label = sprintf("%.2f", loading)),
    size = 2.5
  ) +
  labs(
    title = "Rotated Factor Loadings: Seven-Factor EFA",
    x = "Extracted factor",
    y = "Questionnaire item"
  ) +
  theme_minimal() +
  theme(
    axis.text.y = element_text(size = 8),
    plot.title = element_text(face = "bold")
  )

ggsave(
  "outputs/figures/factor_loading_heatmap.png",
  loading_plot,
  width = 10,
  height = 9,
  dpi = 300
)

print(reliability_table)
print(factorability_table)
```

---

# 11. Data-analysis script

Save as:

> `R/03_data_analysis.R`

```r
source("R/00_setup.R")
source("R/01_clean_and_score.R")

# ------------------------------------------------------------
# Recoding contextual predictors
# ------------------------------------------------------------

analysis_dat <- analysis_dat %>%
  mutate(
    employment_group = case_when(
      employment_status == "Not employed" ~ "Not employed",
      employment_status %in% c(
        "Employed full-time",
        "Employed part-time",
        "Self-employed / freelance"
      ) ~ "Employed",
      TRUE ~ NA_character_
    ),

    employment_group = factor(
      employment_group,
      levels = c("Not employed", "Employed")
    ),

    ai_use_numeric = recode(
      ai_use,
      "Never" = 1,
      "Rarely" = 2,
      "Sometimes" = 3,
      "Often" = 4,
      "Very often" = 5,
      .default = NA_real_
    ),

    study_hours_numeric = recode(
      weekly_study_hours,
      "Less than 5 hours" = 1,
      "5–10 hours" = 2,
      "11–15 hours" = 3,
      "16–20 hours" = 4,
      "More than 20 hours" = 5,
      .default = NA_real_
    ),

    programming_experience_numeric = recode(
      programming_experience,
      "No prior experience" = 0,
      "Basic" = 1,
      "Intermediate" = 2,
      "Advanced" = 3,
      "Professional-level experience" = 4,
      .default = NA_real_
    ),

    study_mode = factor(study_mode),
    programme_area = factor(programme_area)
  )

# ------------------------------------------------------------
# Sample profile
# ------------------------------------------------------------

demographic_variables <- c(
  "age_group",
  "gender",
  "degree_level",
  "programme_area",
  "study_mode",
  "delivery_mode",
  "employment_status",
  "academic_stage",
  "weekly_study_hours",
  "ai_use",
  "programming_experience"
)

sample_profile <- purrr::map_dfr(
  demographic_variables,
  function(variable) {
    analysis_dat %>%
      count(.data[[variable]], name = "n") %>%
      mutate(
        variable = variable,
        percentage = 100 * n / sum(n)
      ) %>%
      rename(category = 1) %>%
      select(variable, category, n, percentage)
  }
)

write_csv(
  sample_profile,
  "outputs/tables/sample_profile.csv"
)

# ------------------------------------------------------------
# Score descriptive statistics and confidence intervals
# ------------------------------------------------------------

score_variables <- c(
  "self_efficacy",
  "task_value",
  "intrinsic_motivation",
  "metacognitive_regulation",
  "effort_regulation",
  "time_management",
  "help_seeking",
  "motivation_composite",
  "srl_strategy_composite"
)

score_descriptives <- analysis_dat %>%
  select(all_of(score_variables)) %>%
  pivot_longer(
    everything(),
    names_to = "score",
    values_to = "value"
  ) %>%
  group_by(score) %>%
  summarise(
    n = sum(!is.na(value)),
    mean = mean(value, na.rm = TRUE),
    sd = sd(value, na.rm = TRUE),
    median = median(value, na.rm = TRUE),
    IQR = IQR(value, na.rm = TRUE),
    standard_error = sd / sqrt(n),
    ci_lower = mean -
      qt(0.975, df = n - 1) * standard_error,
    ci_upper = mean +
      qt(0.975, df = n - 1) * standard_error
  )

write_csv(
  score_descriptives,
  "outputs/tables/score_descriptives.csv"
)

score_plot <- score_descriptives %>%
  filter(score %in% names(subscale_items)) %>%
  ggplot(
    aes(x = reorder(score, mean), y = mean)
  ) +
  geom_point(size = 3) +
  geom_errorbar(
    aes(ymin = ci_lower, ymax = ci_upper),
    width = 0.15
  ) +
  coord_flip() +
  labs(
    title = "Adapted MSLQ Subscale Means",
    x = NULL,
    y = "Mean score with 95% confidence interval"
  ) +
  ylim(1, 5) +
  theme_minimal()

ggsave(
  "outputs/figures/subscale_means_95ci.png",
  score_plot,
  width = 8,
  height = 5.5,
  dpi = 300
)

# ------------------------------------------------------------
# Correlation matrix
# ------------------------------------------------------------

subscale_data <- analysis_dat %>%
  select(all_of(names(subscale_items)))

correlation_matrix <- cor(
  subscale_data,
  use = "pairwise.complete.obs",
  method = "spearman"
)

write.csv(
  correlation_matrix,
  "outputs/tables/subscale_spearman_correlations.csv"
)

# ------------------------------------------------------------
# H3: Employment group comparison
# ------------------------------------------------------------

employment_data <- analysis_dat %>%
  filter(!is.na(employment_group))

employment_ttest <- t.test(
  srl_strategy_composite ~ employment_group,
  data = employment_data,
  var.equal = FALSE,
  conf.level = 0.95
)

employment_effect <- effectsize::hedges_g(
  srl_strategy_composite ~ employment_group,
  data = employment_data,
  pooled_sd = FALSE,
  ci = 0.95
)

capture.output(
  employment_ttest,
  file = "outputs/tables/employment_welch_ttest.txt"
)

write_csv(
  as.data.frame(employment_effect),
  "outputs/tables/employment_hedges_g.csv"
)

# ------------------------------------------------------------
# Exploratory programme-area comparison
# ------------------------------------------------------------

programme_counts <- analysis_dat %>%
  count(programme_area, sort = TRUE)

write_csv(
  programme_counts,
  "outputs/tables/programme_group_counts.csv"
)

# Use only when all retained groups have a defensible sample size.
programme_welch <- oneway.test(
  srl_strategy_composite ~ programme_area,
  data = analysis_dat,
  var.equal = FALSE
)

capture.output(
  programme_welch,
  file = "outputs/tables/programme_welch_anova.txt"
)

programme_posthoc <- rstatix::games_howell_test(
  analysis_dat,
  srl_strategy_composite ~ programme_area
)

write_csv(
  programme_posthoc,
  "outputs/tables/programme_games_howell.csv"
)

# ------------------------------------------------------------
# Primary multiple regression
# Outcome excludes the motivational predictor items
# ------------------------------------------------------------

model_data <- analysis_dat %>%
  select(
    srl_strategy_composite,
    self_efficacy,
    task_value,
    intrinsic_motivation,
    ai_use_numeric,
    study_hours_numeric,
    math_statistics_confidence,
    study_mode,
    employment_group
  ) %>%
  drop_na()

primary_model <- lm(
  srl_strategy_composite ~
    self_efficacy +
    task_value +
    intrinsic_motivation +
    ai_use_numeric +
    study_hours_numeric +
    math_statistics_confidence +
    study_mode +
    employment_group,
  data = model_data
)

capture.output(
  summary(primary_model),
  file = "outputs/tables/primary_regression_summary.txt"
)

# Multicollinearity
vif_results <- car::vif(primary_model)

write.csv(
  as.data.frame(vif_results),
  "outputs/tables/regression_vif.csv"
)

# Heteroscedasticity test
bp_test <- lmtest::bptest(primary_model)

capture.output(
  bp_test,
  file = "outputs/tables/breusch_pagan_test.txt"
)

# Robust HC3 standard errors
robust_coefficients <- lmtest::coeftest(
  primary_model,
  vcov. = sandwich::vcovHC(
    primary_model,
    type = "HC3"
  )
)

robust_table <- broom::tidy(
  robust_coefficients,
  conf.int = TRUE
)

write_csv(
  robust_table,
  "outputs/tables/regression_robust_hc3.csv"
)

# Standardized coefficients
standardized_coefficients <-
  parameters::standardize_parameters(
    primary_model,
    method = "refit"
  )

write_csv(
  as.data.frame(standardized_coefficients),
  "outputs/tables/regression_standardized_coefficients.csv"
)

# Model-level statistics
model_summary <- summary(primary_model)

model_fit_table <- tibble(
  n = nobs(primary_model),
  r_squared = model_summary$r.squared,
  adjusted_r_squared = model_summary$adj.r.squared,
  residual_standard_error = model_summary$sigma,
  model_f = unname(model_summary$fstatistic[1]),
  df_model = unname(model_summary$fstatistic[2]),
  df_residual = unname(model_summary$fstatistic[3])
)

write_csv(
  model_fit_table,
  "outputs/tables/regression_model_fit.csv"
)

# ------------------------------------------------------------
# Regression diagnostic plots
# ------------------------------------------------------------

png(
  "outputs/figures/regression_diagnostics.png",
  width = 1800,
  height = 1800,
  res = 220
)

par(mfrow = c(2, 2))
plot(primary_model)
par(mfrow = c(1, 1))

dev.off()

print(score_descriptives)
print(employment_ttest)
print(employment_effect)
print(summary(primary_model))
print(robust_coefficients)
```

---

# 12. R Markdown report files

Create:

> `reports/Validation_Report.Rmd`

```yaml
---
title: "Validation Report: Adapted Motivated Strategies for Learning Questionnaire"
author: "CS5651 Statistical Inference Research Group"
date: "`r format(Sys.Date(), '%d %B %Y')`"
output:
  pdf_document:
    toc: true
    number_sections: true
fontsize: 11pt
geometry: margin=1in
---
```

At the beginning of the report:

```r
```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo = FALSE,
  warning = FALSE,
  message = FALSE,
  fig.align = "center"
)

source("../R/00_setup.R")
source("../R/01_clean_and_score.R")
source("../R/02_validation_analysis.R")
```
```

Create:

> `reports/Data_Analysis_Report.Rmd`

Use the same YAML, but source `03_data_analysis.R`.

Submit both:

- PDF output;
- `.Rmd` source;
- supporting R scripts.

If PDF rendering fails, install TinyTeX once:

```r
install.packages("tinytex")
tinytex::install_tinytex()
```

---

# 13. Quality gates before submission

Do not submit until every item below is checked.

## Validation Report checklist

- [ ] Actual sample size stated consistently
- [ ] Genuine sampling method stated
- [ ] Exclusion flow included
- [ ] No fabricated pilot or retest
- [ ] Adaptation table included
- [ ] Alpha calculated separately for seven scales
- [ ] Omega reported where estimable
- [ ] Item-total diagnostics reported
- [ ] KMO and Bartlett reported
- [ ] Parallel analysis included
- [ ] EFA solution interpreted
- [ ] No claim of complete/permanent validation
- [ ] Limitations included
- [ ] R code and session information retained

## Data Analysis Report checklist

- [ ] Primary hypotheses stated before results
- [ ] Seven subscale scores defined
- [ ] Motivation and SRL composites kept conceptually separate
- [ ] Sample profile reported
- [ ] Means, SDs, and 95% CIs reported
- [ ] Effect sizes reported
- [ ] Regression assumptions checked
- [ ] HC3 robust errors used if needed
- [ ] Cross-sectional results described as associations
- [ ] Exploratory analyses identified
- [ ] Open-ended responses summarized systematically
- [ ] No unsupported low/medium/high score categories
- [ ] Tables and figures numbered and referenced in text

---

# 14. High-impression practices

These require little extra effort but materially improve the submission.

## Use an analysis decision log

Create:

> `Analysis_Decision_Log.docx`

Record:

- why responses were excluded;
- why variables were grouped;
- why EFA factor count was selected;
- whether any item was removed;
- why a statistical alternative was used;
- whether p-values were adjusted.

## Submit reproducible materials

Include:

- raw data, stored privately;
- anonymized cleaned data;
- `.Rmd` reports;
- R scripts;
- output PDFs;
- codebook;
- `renv.lock`;
- `sessionInfo.txt`.

Generate session information using:

```r
capture.output(
  sessionInfo(),
  file = "outputs/sessionInfo.txt"
)
```

## Report negative or weak findings honestly

An instrument with one weak subscale and a transparent discussion is more academically credible than an unrealistic report claiming every statistic was perfect.

## Keep confirmatory and exploratory results separate

Primary:

- motivation → SRL regression;
- AI usage association;
- employment difference.

Exploratory:

- programme area;
- delivery mode;
- degree level;
- gender;
- academic stage.

## Do not manipulate the scale until reliability appears high

An item should be removed only when supported by:

- weak content relevance;
- weak item-total relationship;
- low loading;
- problematic cross-loading;
- unclear wording;
- expert or participant feedback.

---

# 15. Final submission package for these two reports

Prepare:

```text
01_Validation_Report.pdf
01_Validation_Report.Rmd
02_Data_Analysis_Report.pdf
02_Data_Analysis_Report.Rmd
R/
  00_setup.R
  01_clean_and_score.R
  02_validation_analysis.R
  03_data_analysis.R
data_processed/
  MSLQ_Analysis_Data.csv
  MSLQ_Excluded_Responses.csv
  Codebook.xlsx
outputs/
  tables/
  figures/
sessionInfo.txt
renv.lock
Finalised_Questionnaire.pdf
```

Do not publicly share the raw response file. Submit it only if the lecturer requests it and confirm that it contains no identifying information.

---

# 16. Immediate execution order

Given the submission deadline, use this exact order:

1. Freeze the form and export the final response file.
2. Delete test submissions and document exclusions.
3. Update the finalized questionnaire PDF to match the form.
4. Create the RStudio Project and folder structure.
5. Run `00_setup.R`.
6. Place the raw Excel file in `data_raw/`.
7. Run `01_clean_and_score.R`.
8. Manually inspect the final sample and exclusions.
9. Run `02_validation_analysis.R`.
10. Write and render the Validation Report.
11. Run `03_data_analysis.R`.
12. Write and render the Data Analysis Report.
13. Verify every number against generated tables.
14. Create `renv.lock` and `sessionInfo.txt`.
15. Package the PDFs, Rmd files, scripts, figures, tables, and codebook.

**Next operational input:** the final untouched Google Forms Excel export. Once that file is fixed in the project folder, the column-mapping and analysis scripts can be verified against its exact headings.
