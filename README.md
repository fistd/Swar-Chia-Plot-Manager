## All Commands

##### Example Usage of Commands
```text
> python3 manager.py start

> python3 manager.py restart

> python3 manager.py stop

> python3 manager.py view

> python3 manager.py status

> python3 manager.py analyze_logs
```

### start

This command will start the manager in the background. Once you start it, it will always be running unless all jobs have had their `max_plots` completed or there is an error. Errors will be logged in a file created `debug.log`

### stop

This command will terminate the manager in the background. It does not stop running plots, it will only stop new plots from getting created.

### restart

This command will run start and stop sequentially.

### view

This command will show the view that you can use to keep track of your running plots. This will get updated every X seconds defined by your `config.yaml`.

### status

This command will a single snapshot of the view. It will not loop.

### analyze_logs

This command will analyze all completed plot logs in your log folder and calculate the proper weights and line ends for your computer's configuration. Just populate the returned values under the `progress` section in your `config.yaml`. This only impacts the progress bar.


## Installation

The installation of this library is straightforward. I have attached detailed instructions below that should help you get started. 

#### NOTE: If `python` does not work, please try `python3`.

1. Download and Install Python 3.7 or higher: https://www.python.org/
2. `git clone` this repo or download it.
3. Open CommandPrompt / PowerShell / Terminal and `cd` into the main library folder.
   * Example: `cd C:\Users\Swar\Documents\Swar-Chia-Plot-Manager`
4. OPTIONAL: Create a virtual environment for Python. This is recommended if you use Python for other things.
	1. Create a new python environment: `python -m venv venv`
	   * The second `venv` can be renamed to whatever you want. I prefer `venv` because it's a standard.
	2. Activate the virtual environment. This must be done *every single time* you open a new window.
	   * Example Windows: `venv\Scripts\activate`
	   * Example Linux: `. ./venv/bin/activate` or `source ./venv/bin/activate`
	   * Example Mac OS: `/Applications/Chia.app/Contents/Resources/app.asar.unpacked/daemon/chia`
	3. Confirm that it has activated by seeing the `(venv)` prefix. The prefix will change depending on what you named it.
5. Install the required modules: `pip install -r requirements.txt`
	* If you plan on using Notifications or Prometheus then run the following to install the required modules: `pip install -r requirements-notification.txt`
6. Copy `config.yaml.default` and name it as `config.yaml` in the same directory.
7. Edit and set up the config.yaml to your own personal settings. There is more help on this below.
	* You will need to add the `chia_location` as well! This should point to your chia executable.
9. Run the Manager: `python manager.py start`
   * This will start a process in the background that will manage plots based on your inputted settings.
10. Run the View: `python manager.py view`
   * This will loop through a view screen with details about active plots.


## Configuration

The configuration of this library is unique to every end-user. The `config.yaml` file is where the configuration will live. 

This plot manager works based on the idea of jobs. Each job will have its own settings that you can configure and customize. No two drives are unique so this will provide flexibility for your own constraints and requirements.

### chia_location

This is a single variable that should contain the location of your chia executable file. This is the blockchain executable.

* Windows Example: `C:\Users\<USERNAME>\AppData\Local\chia-blockchain\app-1.1.2\resources\app.asar.unpacked\daemon\chia.exe`
* Linux Example: `/usr/lib/chia-blockchain/resources/app.asar.unpacked/daemon/chia`
* Another Linux Example: `/home/swar/chia-blockchain/venv/bin/chia`

### manager

These are the config settings that will only be used by the plot manager.

* `check_interval` - The number of seconds to wait before checking to see if a new job should start.
* `log_level` - Keep this on ERROR to only record when there are errors. Change this to INFO in order to see more detailed logging. Warning: INFO will write a lot of information.

### log

* `folder_path` - This is the folder where your log files for plots will be saved.

### view

These are the settings that will be used by the view.

* `check_interval` - The number of seconds to wait before updating the view.
* `datetime_format` - The datetime format that you want displayed in the view. See here for formatting: https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes
* `include_seconds_for_phase` - This dictates whether seconds are included in the phase times.
* `include_drive_info` - This dictates whether the drive information will be showed.
* `include_cpu` - This dictates whether the CPU information will be showed.
* `include_ram` - This dictates whether the RAM information will be showed.
* `include_plot_stats` - This dictates whether the plot stats will be showed.

### notifications

These are different settings in order to send notifications when the plot manager starts and when a plot has been completed.

### instrumentation

Settings for enabling Prometheus to gather metrics.

* `prometheus_enabled` - If enabled, metrics will be gathered and an HTTP server will start up to expose the metrics for Prometheus.
* `prometheus_port` - HTTP server port.

List of Metrics Gathered

- **chia_running_plots**: A [Gauge](https://prometheus.io/docs/concepts/metric_types/#gauge) to see how many plots are currently being created.
- **chia_completed_plots**: A [Counter](https://prometheus.io/docs/concepts/metric_types/#counter) for completed plots.

### progress

* `phase_line_end` - These are the settings that will be used to dictate when a phase ends in the progress bar. It is supposed to reflect the line at which the phase will end so the progress calculations can use that information with the existing log file to calculate a progress percent. 
* `phase_weight` - These are the weight to assign to each phase in the progress calculations. Typically, Phase 1 and 3 are the longest phases so they will hold more weight than the others.

### global
* `max_concurrent` - The maximum number of plots that your system can run. The manager will not kick off more than this number of plots total over time.
* `max_for_phase_1` - The maximum number of plots that your system can run in phase 1.
* `minimum_minutes_between_jobs` - The minimum number of minutes before starting a new plotting job, this prevents multiple jobs from starting at the exact same time. This will alleviate congestion on destination drive. Set to 0 to disable.

### job

Each job must have unique temporary directories.

These are the settings that will be used by each job. Please note you can have multiple jobs and each job should be in YAML format in order for it to be interpreted correctly. Almost all the values here will be passed into the Chia executable file. 

Check for more details on the Chia CLI here: https://github.com/Chia-Network/chia-blockchain/wiki/CLI-Commands-Reference

* `name` - This is the name that you want to give to the job.
* `max_plots` - This is the maximum number of jobs to make in one run of the manager. Any restarts to manager will reset this variable. It is only here to help with short term plotting.
* [OPTIONAL]`farmer_public_key` - Your farmer public key. If none is provided, it will not pass in this variable to the chia executable which results in your default keys being used. This is only needed if you have chia set up on a machine that does not have your credentials.
* [OPTIONAL]`pool_public_key` - Your pool public key. Same information as the above. 
* `temporary_directory` - Can be a single value or a list of values. This is where the plotting will take place. If you provide a list, it will cycle through each drive one by one. These directories must be unique from one another.
* [OPTIONAL]`temporary2_directory` - Can be a single value or a list of values. This is an optional parameter to use in case you want to use the temporary2 directory functionality of Chia plotting.
* `destination_directory` - Can be a single value or a list of values. This is the final directory where the plot will be transferred once it is completed. If you provide a list, it will cycle through each drive one by one.  
* `size` - This refers to the k size of the plot. You would type in something like 32, 33, 34, 35... in here.
* `bitfield` - This refers to whether you want to use bitfield or not in your plotting. Typically, you want to keep this as true.
* `threads` - This is the number of threads that will be assigned to the plotter. Only phase 1 uses more than 1 thread.
* `buckets` - The number of buckets to use. The default provided by Chia is 128.
* `memory_buffer` - The amount of memory you want to allocate to the process.
* `max_concurrent` - The maximum number of plots to have for this job at any given time.
* `max_concurrent_with_start_early` - The maximum number of plots to have for this job at any given time including phases that started early.
* `initial_delay_minutes` - This is the initial delay that is used when initiate the first job. It is only ever considered once. If you restart manager, it will still adhere to this value.
* `stagger_minutes` - The amount of minutes to wait before the next plot for this job can get kicked off. You can even set this to zero if you want your plots to get kicked off immediately when the concurrent limits allow for it.
* `max_for_phase_1` - The maximum number of plots on phase 1 for this job.
* `concurrency_start_early_phase` - The phase in which you want to start a plot early. It is recommended to use 4 for this field.
* `concurrency_start_early_phase_delay` - The maximum number of minutes to wait before a new plot gets kicked off when the start early phase has been detected.
* `temporary2_destination_sync` - This field will always submit the destination directory as the temporary2 directory. These two directories will be in sync so that they will always be submitted as the same value.
* `exclude_final_directory` - Whether to skip adding `destination_directory` to harvester for farming. This is a Chia feature.
* `skip_full_destinations` - When this is enabled it will calculate the sizes of all running plots and the future plot to determine if there is enough space left on the drive to start a job. If there is not, it will skip the destination and move onto the next one. Once all are full, it will disable the job.
* `unix_process_priority` - UNIX Only. This is the priority that plots will be given when they are spawned. UNIX values must be between -20 and 19. The higher the value, the lower the priority of the process.
* `windows_process_priority` - Windows Only. This is the priority that plots will be given when they are spawned. Windows values vary and should be set to one of the following values:
	* 16384 `BELOW_NORMAL_PRIORITY_CLASS`
	* 32    `NORMAL_PRIORITY_CLASS`
	* 32768 `ABOVE_NORMAL_PRIORITY_CLASS`
	* 128   `HIGH_PRIORITY_CLASS`
	* 256   `REALTIME_PRIORITY_CLASS`
* `enable_cpu_affinity` - Enable or disable cpu affinity for plot processes. Systems that plot and harvest may see improved harvester or node performance when excluding one or two threads for plotting process.
* `cpu_affinity` - List of cpu (or threads) to allocate for plot processes. The default example assumes you have a hyper-threaded 4 core CPU (8 logical cores). This config will restrict plot processes to use logical cores 0-5, leaving logical cores 6 and 7 for other processes (6 restricted, 2 free).
