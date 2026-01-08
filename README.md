<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Wet-Lab Mosaicism Benchmarking Workflow</title>
  <style>
    body {
      font-family: "Segoe UI", Roboto, Arial, sans-serif;
      line-height: 1.6;
      margin: 40px;
      color: #333;
    }
    h1, h2, h3 {
      color: #1b3a4b;
    }
    pre, code {
      background-color: #f4f4f4;
      padding: 10px;
      border-radius: 6px;
      display: block;
      overflow-x: auto;
      font-family: "Courier New", monospace;
    }
    table {
      border-collapse: collapse;
      margin-top: 10px;
      margin-bottom: 20px;
      width: 100%;
    }
    table, th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #e0f0f8;
    }
    code.inline {
      display: inline;
      padding: 2px 4px;
      border-radius: 4px;
      background-color: #f0f0f0;
    }
  </style>
</head>
<body>

<h1>🧬 Wet-Lab Mosaicism Benchmarking Workflow</h1>

<p>This repository provides a framework to evaluate <strong>mosaic variant detection performance</strong> in
<strong>whole-genome sequencing (WGS)</strong> data using <strong>wet-lab–mixed samples</strong>.</p>

<hr>

<h2>Objective</h2>
<p>To benchmark the <strong>sensitivity</strong> and <strong>specificity</strong> of variant detection tools in identifying
low-frequency (<em>mosaic-like</em>) variants generated from known wet-lab mixtures of two reference genomes
(e.g., NA12878 and NA24385).</p>

<hr>

<h2>Background</h2>
<p>Humans are <strong>diploid</strong>, meaning each position in the genome has two alleles — one from the mother and one from the father.</p>

<table>
  <tr>
    <th>Variant Type</th>
    <th>Definition</th>
    <th>Expected Variant Allele Fraction (VAF)</th>
  </tr>
  <tr>
    <td>Homozygous</td>
    <td>Both alleles mutated</td>
    <td>~100%</td>
  </tr>
  <tr>
    <td>Heterozygous</td>
    <td>One allele mutated</td>
    <td>~50%</td>
  </tr>
</table>

<p>When two genomes are mixed (e.g., <strong>80:20</strong>), variants private to the minor component appear at lower VAFs — mimicking mosaicism.</p>

<table>
  <tr>
    <th>Variant Source</th>
    <th>Genotype</th>
    <th>Expected VAF (80:20 mix)</th>
  </tr>
  <tr>
    <td>A (major, 80%)</td>
    <td>Homozygous</td>
    <td>~1.0</td>
  </tr>
  <tr>
    <td>A (major, 80%)</td>
    <td>Heterozygous</td>
    <td>~0.4–0.5</td>
  </tr>
  <tr>
    <td>B (minor, 20%)</td>
    <td>Homozygous</td>
    <td>~0.2</td>
  </tr>
  <tr>
    <td>B (minor, 20%)</td>
    <td>Heterozygous</td>
    <td>~0.1</td>
  </tr>
</table>

<hr>

<h2>Workflow</h2>

<pre>
Step 1: Input Samples
 ├─ Sample A (e.g., NA12878)
 └─ Sample B (e.g., NA24385)

Step 2: Identify Private Variants (Truth Sets)
 ├─ Call variants separately for A and B
 ├─ Remove shared coordinates
 ├─ Outputs:
 │    - A_private.vcf (unique to A)
 │    - B_private.vcf (unique to B)

Step 3: Wet-Lab Mixing and Sequencing
 ├─ Mix DNA of A and B in known ratio (e.g., 80:20)
 ├─ Sequence mixed DNA → FASTQs
 ├─ Align reads → mixed.bam
 └─ Expected VAFs:
     - A variants: near-normal (0.4–1.0)
     - B variants: low-frequency (0.1–0.2, mosaic-like)

Step 4: Variant Calling on Mixed Sample
 ├─ Use mosaic-sensitive variant caller
 │    (Mutect2, LoFreq, Strelka2, DeepVariant)
 └─ Output: mixed_called.vcf

Step 5: Compare Calls vs Truth Sets
 ├─ For each truth variant:
 │    Found → True Positive (TP)
 │    Missing → False Negative (FN)
 ├─ Variants not in truth sets → False Positives (FP)
 └─ Optional: compute VAF from BAM for validation

Step 6: Compute Metrics
 ├─ Sensitivity = TP / (TP + FN)
 ├─ Specificity = TN / (TN + FP)
 ├─ Precision (PPV) = TP / (TP + FP)
 └─ Report metrics by VAF bins (5%, 10%, 20%)

Step 7: Interpretation
 ├─ Plot sensitivity vs allele fraction
 └─ Identify detection limits for each tool (e.g., reliable down to 10% VAF)
</pre>

<hr>

<h2>Output Metrics</h2>
<ul>
  <li><strong>True Positives (TP):</strong> Simulated low-VAF variants correctly detected.</li>
  <li><strong>False Negatives (FN):</strong> Known low-VAF variants not detected.</li>
  <li><strong>False Positives (FP):</strong> Variants detected but absent from both truth sets.</li>
  <li><strong>True Negatives (TN):</strong> Positions correctly identified as non-variant.</li>
</ul>

<hr>

<h2>Deliverables</h2>
<ul>
  <li>A_private.vcf</li>
  <li>B_private.vcf</li>
  <li>mixed_called.vcf</li>
  <li>metrics_summary.tsv</li>
  <li>plots/
    <ul>
      <li>sensitivity_vs_vaf.png</li>
      <li>precision_recall_curve.png</li>
    </ul>
  </li>
</ul>

<hr>

<h2>Usage</h2>

<h3>1️⃣ Folder Structure</h3>
<pre>
mosaicism-benchmarking/
│
├── data/
│   ├── sampleA.vcf.gz
│   ├── sampleB.vcf.gz
│   ├── mixed_sample.bam
│   └── reference.fa
│
├── scripts/
│   ├── identify_private_variants.sh
│   ├── call_variants.sh
│   ├── compare_vcfs.py
│   ├── compute_metrics.py
│   └── plot_vaf_metrics.R
│
├── results/
│   ├── A_private.vcf
│   ├── B_private.vcf
│   ├── mixed_called.vcf
│   ├── metrics_summary.tsv
│   └── plots/
│       ├── sensitivity_vs_vaf.png
│       └── precision_recall_curve.png
│
└── README.html
</pre>

<h3>2️⃣ Example Commands</h3>

<pre>
# Identify private variants
bash scripts/identify_private_variants.sh data/sampleA.vcf.gz data/sampleB.vcf.gz results/

# Call variants on mixed sample
bash scripts/call_variants.sh data/mixed_sample.bam data/reference.fa results/mixed_called.vcf

# Compare called vs truth sets
python scripts/compare_vcfs.py results/A_private.vcf results/B_private.vcf results/mixed_called.vcf -o results/

# Compute sensitivity/specificity
python scripts/compute_metrics.py results/comparison_output.tsv -o results/metrics_summary.tsv

# Plot performance curves
Rscript scripts/plot_vaf_metrics.R results/metrics_summary.tsv results/plots/
</pre>

<h3>3️⃣ Requirements</h3>
<ul>
  <li>Variant callers: GATK Mutect2, LoFreq, Strelka2, or DeepVariant</li>
  <li>Python 3.8+, bcftools, samtools, bedtools</li>
  <li>R with <code class="inline">ggplot2</code></li>
  <li>Reference genome: hg38 or GRCh37</li>
</ul>

<hr>

<h2>Notes</h2>
<ul>
  <li>Focus initially on <strong>whole-genome sequencing (WGS)</strong>; exome analysis can follow later.</li>
  <li>Minimum recommended coverage: 50–100× for minor allele detection.</li>
  <li>Ensure consistent file naming and indexing (VCF, BAM).</li>
  <li>Can be adapted for <strong>in silico mixtures</strong> as well.</li>
</ul>

</body>
</html>
