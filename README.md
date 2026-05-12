from flask import Flask, render_template_string, request
import math

app = Flask(__name__)

# This is the HTML template with embedded Bootstrap CSS for styling
HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>H.A.R.I.S. qPCR Web Calculator</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background-color: #f8f9fa; padding-top: 2rem; }
        .card { box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 2rem; }
        .highlight { background-color: #e9ecef; font-weight: bold; }
    </style>
</head>
<body>
<div class="container">
    <h2 class="text-center mb-4">H.A.R.I.S. qPCR Calculator (Sample-Based Mix)</h2>

    <div class="row">
        <div class="col-md-4">
            <div class="card">
                <div class="card-header bg-primary text-white">
                    <h5 class="card-title mb-0">Input Parameters</h5>
                </div>
                <div class="card-body">
                    <form method="POST">
                        <div class="mb-2">
                            <label>No. of Patients:</label>
                            <input type="number" class="form-control" name="patients" value="{{ inputs.patients if inputs else 3 }}" required>
                        </div>
                        <div class="mb-2">
                            <label>No. of Controls:</label>
                            <input type="number" class="form-control" name="controls" value="{{ inputs.controls if inputs else 1 }}" required>
                        </div>
                        <div class="mb-2">
                            <label>Replicates (e.g., 3 for Triplicates):</label>
                            <input type="number" class="form-control" name="replicates" value="{{ inputs.replicates if inputs else 3 }}" required>
                        </div>
                        <div class="mb-2">
                            <label>Number of Primers:</label>
                            <input type="number" class="form-control" name="primers" value="{{ inputs.primers if inputs else 3 }}" required>
                        </div>
                        <hr>
                        <div class="mb-2">
                            <label>Master Mix Vol per Well (µL):</label>
                            <input type="number" step="0.1" class="form-control" name="vol_mm" value="{{ inputs.vol_mm if inputs else 5 }}" required>
                        </div>
                        <div class="mb-2">
                            <label>cDNA Vol per Well (µL):</label>
                            <input type="number" step="0.1" class="form-control" name="vol_cdna" value="{{ inputs.vol_cdna if inputs else 4 }}" required>
                        </div>
                        <div class="mb-2">
                            <label>Primer Vol per Well (µL):</label>
                            <input type="number" step="0.1" class="form-control" name="vol_primer" value="{{ inputs.vol_primer if inputs else 1 }}" required>
                        </div>
                        <div class="mb-3">
                            <label>Overage % (optional):</label>
                            <input type="number" step="0.1" class="form-control" name="overage" value="{{ inputs.overage if inputs else 0 }}">
                        </div>
                        <button type="submit" class="btn btn-success w-100">Calculate Volumes</button>
                    </form>
                </div>
            </div>
        </div>

        <div class="col-md-8">
            {% if results %}
            <div class="card">
                <div class="card-header bg-dark text-white">
                    <h5 class="card-title mb-0">Grand Totals</h5>
                </div>
                <div class="card-body row text-center">
                    <div class="col-sm-3">
                        <h3 class="text-primary">{{ results.total_wells }}</h3>
                        <p>Total Tubes/Wells</p>
                    </div>
                    <div class="col-sm-3">
                        <h3 class="text-danger">{{ results.grand_total_mm }} µL</h3>
                        <p>Total Master Mix</p>
                    </div>
                    <div class="col-sm-3">
                        <h3 class="text-success">{{ results.grand_total_cdna }} µL</h3>
                        <p>Total cDNA (All)</p>
                    </div>
                    <div class="col-sm-3">
                        <h3 class="text-warning">{{ results.grand_total_primer }} µL</h3>
                        <p>Total Primer (All)</p>
                    </div>
                </div>
            </div>

            <div class="card">
                <div class="card-header bg-secondary text-white">
                    <h5 class="card-title mb-0">Bench Protocol (Based on your notebook)</h5>
                </div>
                <div class="card-body">
                    <h5>Step 1: Make Sample-Specific Mixes</h5>
                    <p>You have <strong>{{ results.total_samples }} total samples</strong> (Patients + Controls). Prepare a separate tube for EACH sample.</p>
                    <table class="table table-bordered text-center">
                        <thead class="table-light">
                            <tr>
                                <th>For 1 Sample Tube</th>
                                <th>Volume</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr>
                                <td>Master Mix</td>
                                <td>{{ results.mm_per_sample_tube }} µL</td>
                            </tr>
                            <tr>
                                <td>Specific cDNA</td>
                                <td>{{ results.cdna_per_sample_tube }} µL</td>
                            </tr>
                            <tr class="highlight">
                                <td>Total Volume in Tube</td>
                                <td>{{ results.total_vol_per_sample_tube }} µL</td>
                            </tr>
                        </tbody>
                    </table>

                    <h5 class="mt-4">Step 2: Aliquot to PCR Tubes</h5>
                    <p>From each Sample Tube, aliquot <strong>{{ results.aliquot_per_well }} µL</strong> into <strong>{{ results.wells_per_sample }} PCR tubes</strong> ({{ results.primers }} primers × {{ results.replicates }} replicates).</p>

                    <h5 class="mt-4">Step 3: Add Primers</h5>
                    <p>Finally, add <strong>{{ results.vol_primer }} µL</strong> of the specific primer to the respective tubes.</p>
                    <p class="text-muted small">Note: To run the entire plate, you will consume {{ results.total_specific_primer_needed }} µL of EACH specific primer.</p>
                </div>
            </div>
            {% else %}
            <div class="alert alert-info text-center mt-5">
                Enter your parameters and click calculate to generate your prep data.
            </div>
            {% endif %}
        </div>
    </div>
</div>
</body>
</html>
"""


@app.route('/', methods=['GET', 'POST'])
def index():
    results = None
    inputs = None

    if request.method == 'POST':
        # 1. Get user inputs
        inputs = {
            'patients': int(request.form.get('patients', 0)),
            'controls': int(request.form.get('controls', 0)),
            'replicates': int(request.form.get('replicates', 0)),
            'primers': int(request.form.get('primers', 0)),
            'vol_mm': float(request.form.get('vol_mm', 0)),
            'vol_cdna': float(request.form.get('vol_cdna', 0)),
            'vol_primer': float(request.form.get('vol_primer', 0)),
            'overage': float(request.form.get('overage', 0))
        }

        # 2. Math Logic (With optional overage multiplier)
        multiplier = 1 + (inputs['overage'] / 100)

        total_samples = inputs['patients'] + inputs['controls']
        wells_per_sample = inputs['primers'] * inputs['replicates']

        # Apply overage factor
        calc_wells_per_sample = math.ceil(wells_per_sample * multiplier)
        total_wells_needed = total_samples * calc_wells_per_sample

        # Sample Tube Math (Step 1)
        mm_per_sample_tube = round(calc_wells_per_sample * inputs['vol_mm'], 2)
        cdna_per_sample_tube = round(calc_wells_per_sample * inputs['vol_cdna'], 2)
        total_vol_per_sample_tube = round(mm_per_sample_tube + cdna_per_sample_tube, 2)

        # Grand Totals
        grand_total_mm = round(mm_per_sample_tube * total_samples, 2)
        grand_total_cdna = round(cdna_per_sample_tube * total_samples, 2)

        # Primer Math
        rxns_per_specific_primer = math.ceil((total_samples * inputs['replicates']) * multiplier)
        total_specific_primer_needed = round(rxns_per_specific_primer * inputs['vol_primer'], 2)
        grand_total_primer = round(total_specific_primer_needed * inputs['primers'], 2)

        # 3. Package results for HTML
        results = {
            'total_samples': total_samples,
            'wells_per_sample': calc_wells_per_sample,
            'total_wells': total_wells_needed,
            'primers': inputs['primers'],
            'replicates': inputs['replicates'],
            'vol_primer': inputs['vol_primer'],
            'mm_per_sample_tube': mm_per_sample_tube,
            'cdna_per_sample_tube': cdna_per_sample_tube,
            'total_vol_per_sample_tube': total_vol_per_sample_tube,
            'aliquot_per_well': round(inputs['vol_mm'] + inputs['vol_cdna'], 2),
            'total_specific_primer_needed': total_specific_primer_needed,
            'grand_total_mm': grand_total_mm,
            'grand_total_cdna': grand_total_cdna,
            'grand_total_primer': grand_total_primer
        }

    return render_template_string(HTML_TEMPLATE, results=results, inputs=inputs)


if __name__ == '__main__':
    # Runs on localhost URL: http://127.0.0.1:5000
    app.run(debug=True, port=5000)
