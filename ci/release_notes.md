# Kakfa v1.0.0

The deployment manifest `manifests/kafka-service-broker.yml` now deploys Apache Kafka v1.0.0.

## Different upstream releases

The deployemnt manifest has also switched from bigdata-boshrelease to using two smaller, dedicated releases:

* https://github.com/cppforlife/zookeeper-release
* https://github.com/cloudfoundry-community/kafka-boshrelease
