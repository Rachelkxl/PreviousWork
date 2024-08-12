# 1. Project: Wastewater Flow Forecasting
Pattern Analysis
- Description: Generated and visualized weekly, monthly, and seasonal wastewater flow patterns as a key data source for teams' publications.
- Code Snippet (Python):
The piece of code determines the outliers using three-standard deviation rule.

```python
# Determine the outliers
def get_median_filtered(signal, threshold):
    signal = signal.copy()
    difference = np.abs(signal - np.median(signal))
    median_difference = np.median(difference)
    if median_difference == 0:
        s = 0
    else:
        s = difference / float(median_difference)
    mask = s > threshold
    signal[mask] = np.median(signal)
    return signal

data_hour['Find_Outliers'] = get_median_filtered(data_hour['Flow'].values, threshold=3)
outliers_hour = data_hour[data_hour['Find_Outliers'] != data_hour['Flow']]
```
Wastewater Prediction Model
- Description: Developed a wastewater prediction model using a random forest algorithm, including data preprocessing, model training & testing, performance evaluation, and results visualization.
- Code Snippet (Python):
The piece of code focuses on data training processes.

```python
# train model
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import RandomizedSearchCV

forest_reg = RandomForestRegressor(criterion='mse', min_samples_leaf=1)

param_dist = {'n_estimators': [300,1000,3000], 
               'max_features':['auto','sqrt','log2']}
# randomized search
n_iter_search = 9
random_search = RandomizedSearchCV(forest_reg, 
                                   param_distributions=param_dist,
                                   n_iter=n_iter_search,
                                   scoring='neg_mean_squared_error',
                                   cv=3,
                                   random_state=43,
                                   refit=True)

random_search.fit(train_features, train_labels)


# show the best estimator:
forest_reg = random_search.best_estimator_
forest_reg.fit(train_features, train_labels)


## make predictions on the date set
predictions_test = forest_reg.predict(test_features)
predictions_train = forest_reg.predict(train_features)
```

# 2.Project: Automated Code Compilation Bash Script and Pipeline


- Code Snippet (Bash):
This piece of code checks for the existence of a flag file, which acts as a lock that controls access to the network gateway.

```bash
ls $flag_dir/flag_*

# Check if the flag file exists
if [[ $? == 0 ]]
then
	# Retrieve a list of filenames removing the "flag_" prefix
	# i.e. process IDs
	flags=($(ls $flag_dir/flag_* | cut -c31-))
	for i in ${flags[@]}
	do
		ps -p $i
		if [[ $? != 0 ]]
		then 
			# Clean the corresponding flag file if the process does not exist
			rm -f $flag_dir/flag_$i
			echo "Process $i does not exist, file flag_$i cleaned"
		fi
	done
fi
```

```groovy
void create_lock() {
    pid = sh(script: "echo \$PPID", returnStdout: true).trim()
    while (!(fileExists("${flag_dir}/flag_${pid}"))) {
        sleep 5
        lock_exists = sh(script: "ls ${flag_dir}/flag_*", returnStatus: true)
        if (lock_exists != 0) {
            sh "touch ${flag_dir}/flag_${pid}"
            sleep 2
            flags = sh(script: "ls ${flag_dir}/flag_* | cut -c31-", returnStdout: true).trim()
            flags = flags.split("\n")
            flags_num = []
            for (i in flags) {
                flags_num.add(evaluate(i))
            }
            min = flags_num.min()
            if (evaluate(pid) != flags_num.min()) {
                sh "rm -f ${flag_dir}/flag_${pid}"
            }
        }
    }
    date = new Date()
    sh "echo ${date} >> ${flag_dir}/lock_flags.log"
    sh "echo ${JOB_NAME}_${BUILD_NUMBER}: PID ${pid} get the lock >> ${flag_dir}/lock_flags.log"
}

void release_lock() {
    if (fileExists("${flag_dir}/flag_${pid}")) {
        sh "rm -f ${flag_dir}/flag_${pid}"
    }
    date = new Date()
    sh "echo ${date} >> ${flag_dir}/lock_flags.log"
    sh "echo PID ${pid} relase the lock >> ${flag_dir}/lock_flags.log"
    sh "echo '' >> ${flag_dir}/lock_flags.log"
}
```

