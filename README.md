REDAME SIMPLE LIGHTGBM
# tracking_ad
this repo implement my recent kaggle ct with simple lightgbm model and memory trick 

# memory status
* process = psutil.Process(os.getpid())
* memused = process.memory_info().rss
* print('Total memory in use before reading data: {:.02f} GB '
      ''.format(memused / (2 ** 30)))

# read data
* df_train = pd.read_hdf('../data/train_v2.hdf')
# col
* target = 'is_attributed'
* features = [
    'app',
    'device',
    'os',
    'channel',
    'hour',
    'in_test_hh',
    'ip_day_hour_clicks',
    'ip_app_day_hour_clicks',
    'ip_os_day_hour_clicks',
    'ip_device_day_hour_clicks',
    'ip_channel_day_hour_clicks',
    'app_day_hour_clicks',
    'app_device_day_hour_clicks',
    'app_channel_day_hour_clicks',
    'ip_app_day_clicks',
    'ip_os_day_clicks',
    'ip_device_day_clicks',
    'app_os_day_clicks',
    'app_device_day_clicks',
    'ip_app_device_day_clicks',
    'ip_app_os_device_day_clicks',
    'ip_day_test_hh_clicks',
]
# categorical
* categorical_features = [
    'app',
    'device',
    'os',
    'channel',
    'hour',
    'in_test_hh',
]
# prep data
* dtrain = lightgbm.Dataset(
    df_train[features].values,
    label=df_train[target].values,
    feature_name=features,
    categorical_feature=categorical_features,
    free_raw_data=False,
)
* del df_train
* gc.collect()
* print('done data prep!!!')
# memory status
* memused = process.memory_info().rss
* print('Total memory in use after reading data: {:.02f} GB '
      ''.format(memused / (2 ** 30)))

t0 = time.time()
###################################################################
* params = {
    'boosting_type': 'gbdt',
    'objective': 'binary',
    'learning_rate': 0.03,
    'num_leaves': 12,
    'max_depth': 4,
    'min_split_gain': 0,
    'subsample': 0.9,
    'subsample_freq': 1,
    'colsample_bytree': 0.9,
    'min_child_samples': 100,
    'min_child_weight': 0,
    'max_bin': 100,
    'subsample_for_bin': 200000,
    'reg_alpha': 0,
    'reg_lambda': 0,
    'scale_pos_weight': 100,
    'metric': 'auc',
    'nthread': 22,
    'verbose': 0,
}
# train
model = lightgbm.train(
    params=params,
    train_set=dtrain,
    num_boost_round=2800,
    feature_name=features,
    categorical_feature=categorical_features,
    verbose_eval=1,
    # init_model='model_v2.txt'
)

* t1 = time.time()
* t_min = np.round((t1-t0) / 60, 2)
print('It took {} mins to train model'.format(t_min))
# save model
# model.save_model('model_v2.txt')
# clean up
del dtrain
gc.collect()

# submit
df_test = pd.read_hdf('../data/test_v2.hdf')
# pred
df_test['is_attributed'] = model.predict(df_test[features])
# create submission
sub_cols = ['click_id', 'is_attributed']
df_sub = df_test[sub_cols]
# save
df_sub.to_csv('../data/submission_v2.csv', index=False)
