```python
FROM m.daocloud.io/docker.io/library/python:3-alpine  
LABEL authors="yedongdong"  
  
RUN mkdir /app  
WORKDIR /app  
COPY requirements.txt ./  
RUN pip install --no-cache-dir -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple -r requirements.txt  
RUN echo 'alias mqtt="python /app/mqtt.py"' >> /etc/profile  
ENV ENV=/etc/profile  
COPY . .  
ENTRYPOINT ["python", "/app/mqtt.py"]
```
