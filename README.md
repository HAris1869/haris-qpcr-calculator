# haris-qpcr-calculator
H.A.R.I.S. is a Flask-based web application designed to automate qPCR experimental setup, instantly calculating master mixes, handling overages, and generating sample-specific bench protocols.
# H.A.R.I.S. (Hybrid Analytical Research Integrated Solution) 🧪💻

H.A.R.I.S. is a custom web application designed to streamline and automate qPCR (Quantitative Polymerase Chain Reaction) experimental planning for molecular biology laboratories. 

By replacing manual, error-prone notebook math with an instant computational algorithm, this tool helps researchers calculate precise reagent volumes, manage pipetting overages, and generate step-by-step bench protocols.

## ✨ Features
* **Instant Reagent Calculations:** Calculates the exact grand totals needed for Master Mix, cDNA, Nuclease-Free Water, and specific primers based on user input.
* **Dynamic Bench Protocols:** Automatically generates a sample-specific pipetting guide tailored to the exact number of patients, controls, replicates, and primers.
* **Smart Overage Management:** Includes a customizable overage percentage calculator to account for pipetting errors without wasting expensive reagents.
* **User-Friendly Interface:** Built with a clean, responsive web interface so researchers can access it from any lab computer or tablet.

## 🛠️ Tech Stack
* **Backend:** Python, Flask
* **Frontend:** HTML5, CSS3 (Bootstrap 5)
* **Deployment:** [List your deployment platform here, e.g., PythonAnywhere / Render / Railway]

## 🚀 How to Run Locally

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/your-username/haris-qpcr.git](https://github.com/your-username/haris-qpcr.git)
   cd haris-qpcr
