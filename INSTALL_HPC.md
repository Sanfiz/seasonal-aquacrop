# AquaCrop Installation and Validation on HPC Systems

This guide provides step-by-step instructions for installing, compiling, and validating AquaCrop (from KUL-RSDA/AquaCrop) on High-Performance Computing (HPC) systems without sudo access, GUI, and using Slurm job scheduler.

## Prerequisites

- Access to an HPC cluster with Slurm workload manager
- User account without sudo privileges
- No GUI access (command-line only environment)
- Available modules: git, gcc/g++, make

## Installation Steps

### 1. Clone the Repository

First, clone the AquaCrop repository from KUL-RSDA:

```bash
# Clone the AquaCrop repository
git clone https://github.com/KUL-RSDA/AquaCrop.git
cd AquaCrop
```

### 2. Load Required Modules

Load the necessary modules on your HPC system. Module names may vary depending on your HPC environment:

```bash
# Load git and GCC compiler modules
module load git
module load gcc

# Or, if your system uses versioned modules:
# module load git/2.30
# module load gcc/9.3.0

# Verify loaded modules
module list
```

### 3. Compile AquaCrop

Navigate to the source directory and compile:

```bash
# Navigate to the source directory
cd src

# Compile the source code
make

# Verify compilation was successful
echo "Compilation exit code: $?"
```

### 4. Verify Build Artifacts

After successful compilation, verify that the required artifacts have been created:

```bash
# Check for the aquacrop executable
file aquacrop
ls -lh aquacrop

# Check for the shared library
file libaquacrop.so
ls -lh libaquacrop.so

# Check library dependencies
ldd aquacrop
ldd libaquacrop.so
```

Expected output examples:
- `aquacrop`: ELF 64-bit LSB executable
- `libaquacrop.so`: ELF 64-bit LSB shared object

### 5. Run Test Case

Navigate to the testcase directory and run a validation test:

```bash
# Navigate to testcase directory
cd ../testcase

# Create symbolic link to the aquacrop executable
ln -sf ../src/aquacrop aquacrop

# Verify the symbolic link
ls -l aquacrop

# Run the test case
./aquacrop
```

### 6. Validate Test Results

After running the test case, compare the output with reference files:

```bash
# Check that output files were generated
ls -l OUTP/*.OUT

# Compare output with reference, ignoring the first line
# (first line often contains timestamps or version info)
for outfile in OUTP/*.OUT; do
    reffile="OUTP_REF/$(basename $outfile)"
    if [ -f "$reffile" ]; then
        echo "Comparing $outfile with $reffile"
        diff <(tail -n +2 "$outfile") <(tail -n +2 "$reffile")
        if [ $? -eq 0 ]; then
            echo "✓ $outfile matches reference"
        else
            echo "✗ $outfile differs from reference"
        fi
    fi
done
```

## Running AquaCrop via Slurm

### Basic Slurm Job Script

Create a file named `run_aquacrop.sbatch`:

```bash
#!/bin/bash
#SBATCH --job-name=aquacrop_test
#SBATCH --output=aquacrop_%j.out
#SBATCH --error=aquacrop_%j.err
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=4G

# Load required modules
module load gcc

# Set working directory
cd $SLURM_SUBMIT_DIR

# Print job information
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"
echo "Start time: $(date)"
echo "Working directory: $(pwd)"
echo ""

# Run AquaCrop testcase
cd testcase

# Ensure symbolic link exists
ln -sf ../src/aquacrop aquacrop

# Run the simulation
echo "Running AquaCrop..."
./aquacrop

# Check exit status
EXIT_CODE=$?
echo ""
echo "AquaCrop exit code: $EXIT_CODE"

# Validate outputs
echo ""
echo "Validating outputs..."

# Check if output files were created
if [ -d "OUTP" ]; then
    OUTPUT_COUNT=$(ls -1 OUTP/*.OUT 2>/dev/null | wc -l)
    echo "Generated $OUTPUT_COUNT output files in OUTP/"
    ls -lh OUTP/*.OUT
else
    echo "ERROR: OUTP directory not found!"
    exit 1
fi

# Compare with reference outputs (ignoring first line)
echo ""
echo "Comparing with reference outputs..."
DIFF_COUNT=0
MATCH_COUNT=0

for outfile in OUTP/*.OUT; do
    reffile="OUTP_REF/$(basename $outfile)"
    if [ -f "$reffile" ]; then
        echo "Checking $(basename $outfile)..."
        if diff <(tail -n +2 "$outfile") <(tail -n +2 "$reffile") > /dev/null 2>&1; then
            echo "  ✓ Matches reference"
            MATCH_COUNT=$((MATCH_COUNT + 1))
        else
            echo "  ✗ Differs from reference"
            DIFF_COUNT=$((DIFF_COUNT + 1))
        fi
    else
        echo "  ⚠ No reference file found: $reffile"
    fi
done

echo ""
echo "Validation summary:"
echo "  Files matching reference: $MATCH_COUNT"
echo "  Files differing from reference: $DIFF_COUNT"
echo ""
echo "End time: $(date)"

# Exit with appropriate code
if [ $DIFF_COUNT -gt 0 ]; then
    echo "WARNING: Some outputs differ from reference"
    exit 1
elif [ $EXIT_CODE -ne 0 ]; then
    echo "ERROR: AquaCrop failed with exit code $EXIT_CODE"
    exit $EXIT_CODE
else
    echo "SUCCESS: All validations passed"
    exit 0
fi
```

### Submit the Job

```bash
# Submit the job to Slurm
sbatch run_aquacrop.sbatch

# Check job status
squeue -u $USER

# View output logs after job completion
# (replace JOBID with your actual job ID)
cat aquacrop_JOBID.out
cat aquacrop_JOBID.err
```

### Advanced Slurm Script with Array Jobs

For running multiple scenarios in parallel:

```bash
#!/bin/bash
#SBATCH --job-name=aquacrop_array
#SBATCH --output=aquacrop_%A_%a.out
#SBATCH --error=aquacrop_%A_%a.err
#SBATCH --time=01:00:00
#SBATCH --array=1-10
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --mem=4G

module load gcc

cd testcase

echo "Array Task ID: $SLURM_ARRAY_TASK_ID"
echo "Array Job ID: $SLURM_ARRAY_JOB_ID"

# Your scenario-specific logic here
# Example: different input files or parameters per array task

ln -sf ../src/aquacrop aquacrop
./aquacrop

# Validation logic...
```

## Troubleshooting

### Compilation Errors

If compilation fails:

1. Check that GCC is properly loaded:
   ```bash
   gcc --version
   g++ --version
   ```

2. Check for missing dependencies:
   ```bash
   module avail  # List available modules
   ```

3. Review error messages in the make output

### Runtime Errors

If AquaCrop fails to run:

1. Verify the executable has proper permissions:
   ```bash
   chmod +x src/aquacrop
   ```

2. Check for missing shared libraries:
   ```bash
   ldd src/aquacrop
   ```

3. Ensure input files are in the correct location

### Output Validation Failures

If outputs differ from reference:

1. Check the specific differences:
   ```bash
   diff OUTP/filename.OUT OUTP_REF/filename.OUT
   ```

2. Verify you're using the correct version of AquaCrop

3. Check for floating-point precision differences (may be acceptable)

## Additional Notes

- The first line of output files is typically skipped in comparisons because it often contains timestamps, version numbers, or system-specific information that varies between runs
- Adjust Slurm parameters (time, memory, CPUs) based on your specific simulation requirements
- Check your HPC system's documentation for specific module names and available resources
- Some HPC systems may require additional configuration for shared libraries (e.g., setting `LD_LIBRARY_PATH`)

## References

- AquaCrop repository: https://github.com/KUL-RSDA/AquaCrop
- Slurm documentation: https://slurm.schedmd.com/
