sudo yum install -y git

sudo pip3 install tensorflow-datasets==1.3.2
sudo pip3 install h5py
sudo pip3 install packaging



git clone https://github.com/yahoo/TensorFlowOnSpark.git --branch v1.4.4
pushd TensorFlowOnSpark
zip -r tfspark.zip tensorflowonspark
hdfs dfs -put tfspark.zip
popd



mkdir ${HOME}/mnist
pushd ${HOME}/mnist >/dev/null
curl -O "http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz"
curl -O "http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz"
curl -O "http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz"
curl -O "http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz"
zip -r mnist.zip *
popd >/dev/null

aws s3 cp s3://dongaws-ml/tensorflow/hadoop/tensorflow-hadoop-1.14.0.jar .


spark-submit --jars /${HOME}/tensorflow-hadoop-1.14.0.jar --archives hdfs:///user/hadoop/tfspark.zip#tensorflowonspark,mnist/mnist.zip#mnist TensorFlowOnSpark/examples/mnist/mnist_data_setup.py --output mnist/csv --format csv


spark-submit --py-files TensorFlowOnSpark/tfspark.zip,TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py --num-executors 3  --executor-cores 1 --conf spark.dynamicAllocation.enabled=false --conf spark.executorEnv.LD_LIBRARY_PATH=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/lib/amd64/server/:/usr/lib/hadoop/lib/native/ TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py --images mnist/csv/train/images --labels mnist/csv/train/labels --mode train --model mnist_model
