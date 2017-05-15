# Twitter Sentiment Analysis Example


Register the `python-http` processor and the `jython-processor`

```
dataflow:>app register --type processor --name python-http --uri https://raw.githubusercontent.com/dturanski/spring-cloud-stream-binaries/master/binaries/python-http-processor-rabbit-1.2.1.BUILD.jar
dataflow:>app register --type processor --name jython-processor --uri file:///Users/dturanski/dev/spring-cloud-stream-app-starters-python/apps/python-jython-processor-rabbit/target/python-jython-processor-rabbit-1.2.1.BUILD-SNAPSHOT.jar
```
```


Create the stream. Note the inverted use of single and double quotes. 
```
dataflow:>stream create tweets --definition 'twitterstream --twitter.credentials.consumerKey=fGOCqR0UIXSdQ4SJBB5opCnTQ --twitter.credentials.consumerSecret=ybsF6WPA2hDUV4zjsWgn2p5HLHfWff2RVrmuKlSI0g08UopwDj --twitter.credentials.accessToken=18240547-gk4UePnbXvKmgGrS0zfPwvHtQichBsiBTZU55xdoy --twitter.credentials.accessTokenSecret=G8CXbweKtBKP7NrVJHKZxrLAFYtWbhbwL4clitmJTvTpD | aggregator --aggregator.correlation=1 --aggregator.release=#this.size()==5 | python-http --git.uri=https://github.com/dturanski/python-apps --wrapper.script=test-wrappers/get-tweet-sentiments.py  --httpclient.http-method=POST --httpclient.headersExpression={"Content-Type":"application/json"} --httpclient.url=http://sentiment-compute.cfapps.pez.pivotal.io/polarity_compute | splitter |  log'

dataflow:>stream create sentiments --definition ":tweets.splitter > jython-processor --jython.script=test-wrappers/map-tweet-sentiments.py --jython.variables=neutral=0.45,positive=0.55 --git.uri=https://github.com/dturanski/python-apps | field-value-counter --field-value-counter.name=sentiments --field-value-counter.field-name=sentiment"

```


