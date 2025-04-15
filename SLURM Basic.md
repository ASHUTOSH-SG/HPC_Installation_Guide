
# SLURM Basic Commands & Script Notes

## SLURM Commands

| Command                         | Description                                 |
|----------------------------------|---------------------------------------------|
| `sinfo`                         | View nodes and their states                 |
| `squeue`                        | Show the current job queue                  |
| `sbatch script_name.sh`         | Submit a job using the given script         |
| `scancel job_id`                | Cancel the job with the specified job ID    |
| `sacct`                         | Show history of all jobs                    |
| `sacct -j job_id`               | Show history of a particular job ID         |

---

## SLURM Script Template

```bash
#!/bin/bash
#SBATCH --job-name=test       # Name of the job
#SBATCH --output=%J.out       # Standard output file (%J will be replaced by job ID)
#SBATCH --error=%J.err        # Standard error file
#SBATCH --nodes=1             # Number of nodes required
#SBATCH --ntasks-per-node=1   # Number of tasks per node
#SBATCH --time=01:00:00       # Walltime limit (hh:mm:ss)
#SBATCH --partition=vector    # Partition to submit the job to

# Load environment and activate conda
source ~/.bashrc
conda activate your_conda_env_name

# Run your script
python3 /path/to/your/file.py
```

**Note:** SLURM script file extension should be `.sh`, for example: `slurm_script.sh`

---





## Troubleshooting SLURM Job Issues

### Job Not Starting / Stuck in Queue
- Check resources requested: Use `sinfo` to see available nodes/partitions.
- Check partition name: Make sure `--partition=vector` is a valid partition.
- Node constraints: Try reducing node/task/time requirements.
- Check job status: Use `squeue -u your_username`.

### `conda: command not found` Error
- Add `source ~/.bashrc` before `conda activate` to ensure conda is loaded.

### Python Script Fails Immediately
- Check `%J.err` file for errors.
- Ensure the path `/path/to/your/file.py` is correct and the script has execution permissions (`chmod +x file.py` if needed).
- Check if the conda environment includes required Python packages.

### `Permission denied` Error
- Check script and file permissions: `chmod +x slurm_script.sh`.
- Make sure your script and target Python file are readable and executable.

### Job Runs But Output Is Missing
- Output might be redirected to another file: check `%J.out`.
- Add debug `echo` statements in the script to trace execution steps.

### Job Cancelled Immediately
- Invalid resource request: Try adjusting `--time`, `--nodes`, or `--ntasks`.
- Resource limits exceeded: Check logs or talk to admin if cluster limits apply.

---



## Tips

- Use `%J` in output/error fields to automatically insert the Job ID.
- `conda activate` works only if your environment is properly configured in `.bashrc`.
- Make sure the Python script path is correct and executable.
- Adjust `--time` and `--partition` according to the cluster’s configuration.


- Always test your Python script locally before submitting via SLURM.
- Use `sacct` to check job start/end times and exit status.
- Use `--mail-type=ALL` and `--mail-user=your@email.com`
  to receive email notifications about job status.


