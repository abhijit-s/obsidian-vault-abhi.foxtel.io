```jupyter

print ("Hello");

```





```jupyter
import redis  
  
redis = redis.Redis(  
host= 'localhost',  
port= '6379')  
  
redis.set('mykey', 'Hello from Python!')  
value = redis.get('mykey')  
print(value)

```