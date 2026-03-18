# sls log query

```python
import math
from aliyun.log import *
from aliyun.log.auth import AUTH_VERSION_4

import os
import pytz
from datetime import datetime, timedelta


def get_sls_client(endpoint="cn-hangzhou.log.aliyuncs.com", region='cn-hangzhou'):
    """

    :param endpoint:
    :param region:
    :return:
    """
    access_key_id = os.environ.get('ALIBABA_CLOUD_ACCESS_KEY_ID', 'xxx')
    access_key_secret = os.environ.get('ALIBABA_CLOUD_ACCESS_KEY_SECRET', 'xxxx')
    client = LogClient(endpoint, access_key_id, access_key_secret, auth_version=AUTH_VERSION_4, region=region)
    return client

def load_pod_sls_log(project, logstore, pod_name, date_range=1, dateFrom:str=None, dateTo:str=None, page_size=1000):
    """

    :param project:
    :param logstore:
    :param pod_name:
    :param date_range:
    :param dateFrom:
    :param dateTo:
    :param page_size:
    :return:
    """
    client = get_sls_client()
    query = f'_pod_name_:{pod_name}'
    fromTime = dateFrom if dateFrom else (datetime.now(tz=pytz.timezone("Asia/Shanghai")) - timedelta(days=date_range)).strftime('%Y%m%d %H:%M:%S')

    toTime = dateTo if dateTo else datetime.now(tz=pytz.timezone("Asia/Shanghai")).strftime('%Y%m%d %H:%M:%S')

    print("load sls log for %s: %s from %s to %s with page_size: %s" % (project, logstore, fromTime, toTime, page_size))
    request = GetHistogramsRequest(project=project, logstore=logstore, query=query, fromTime=fromTime,
                                   toTime=toTime)
    response = client.get_histograms(request)
    histograms = response.histograms
    histogram_items = list(filter(lambda x: x.count > 0, histograms))
    for item in histogram_items:
        if item.count < page_size:
            request_ = GetLogsRequest(project, logstore, item.fromTime, item.toTime, '', query=query, line=page_size,
                                      offset=0, reverse=False)
            response_ = client.get_logs(request_)
            for log in response_.get_logs():
                items = log.get_contents()
                print(items['content'])
        else:
            split = math.ceil(item.count / page_size)
            print("------------------ %s ======= %s" % (item.count, split))
            for page in range(split + 1):
                request_ = GetLogsRequest(project, logstore, item.fromTime, item.toTime, '', query=query, line=page_size,
                                          offset=page * page_size, reverse=False)
                response_ = client.get_logs(request_)
                for log in response_.get_logs():
                    items = log.get_contents()
                    print("page: %s ------------ %s " % (page, items['content']))


load_pod_sls_log(project="k8s-log-xx", logstore="k8s-stdout", pod_name="airflow-task-sla-xxx-2-dxm9q")

```


