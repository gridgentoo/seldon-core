# Prediction API

Seldon Core uses REST and gRPC APIs exposed externally for business applications to connect to and also internally for microservices to implement models, routers, combiners and transformers.

 - [External Prediction API](external-prediction.md)
   - Read this if you want to connect external business applications
 - [Internal Prediction API](internal-api.md)
   - Read this if you want to build a microservice to wrap a model or build another type of component such as a router, combiner or transformer


## Proto Buffer and gRPC Definition

The proto file can be found [here](https://github.com/SeldonIO/seldon-core/blob/master/proto/prediction.proto) and is reproduced below:

```proto
syntax = "proto3";

import "google/protobuf/any.proto";
import "google/protobuf/struct.proto";
import "google/protobuf/empty.proto";
import "tensorflow/core/framework/tensor.proto";

package seldon.protos;

option java_package = "io.seldon.protos";
option java_outer_classname = "PredictionProtos";
option go_package = "github.com/seldonio/seldon-core/incubating/wrappers/s2i/go/pkg/api";

// [START Messages]

message SeldonMessage {

  Status status = 1;
  Meta meta = 2;
  oneof data_oneof {
    DefaultData data = 3;
    bytes binData = 4;
    string strData = 5;
    google.protobuf.Value jsonData = 6;
    google.protobuf.Any customData = 7;
  }
}

message DefaultData {
  repeated string names = 1;
  oneof data_oneof {
    Tensor tensor = 2;
    google.protobuf.ListValue ndarray = 3;
    tensorflow.TensorProto tftensor = 4;
  }
}

message Tensor {
  repeated int32 shape = 1 [packed=true];
  repeated double values = 2 [packed=true];
}

message Meta {
  string puid = 1;
  map<string,google.protobuf.Value> tags = 2;
  map<string,int32> routing = 3;
  map<string,string> requestPath = 4;
  repeated Metric metrics = 5;
}

message Metric {
 enum MetricType {
     COUNTER = 0;
     GAUGE = 1;
     TIMER = 2;
 }
 string key = 1;
 MetricType type = 2;
 float value = 3;
 map<string,string> tags = 4;
}

message SeldonMessageList {
  repeated SeldonMessage seldonMessages = 1;
}

message Status {

    enum StatusFlag {
        SUCCESS = 0;
        FAILURE = 1;
    }

    int32 code = 1;
    string info = 2;
    string reason = 3;
    StatusFlag status = 4;
}

message Feedback {
  SeldonMessage request = 1;
  SeldonMessage response = 2;
  float reward = 3;
  SeldonMessage truth = 4;
}

message RequestResponse {
  SeldonMessage request = 1;
  SeldonMessage response = 2;
}


message SeldonModelMetadataRequest
{
  string name = 1;
}


message SeldonMessageMetadata
{
  // SeldonMessage Metadata fields
  string messagetype = 1;
  google.protobuf.Value schema = 2;

  // v2 tensor metadata fields
  string name = 3;
  string datatype = 4;
  repeated int64 shape = 5;
}


message SeldonModelMetadata
{
  string name = 1;
  repeated string versions = 2;
  string platform = 3;
  repeated SeldonMessageMetadata inputs = 4;
  repeated SeldonMessageMetadata outputs = 5;
}


message SeldonGraphMetadata
{
  string name = 1;
  map<string, SeldonModelMetadata> models = 2;

  repeated SeldonMessageMetadata inputs = 3;
  repeated SeldonMessageMetadata outputs = 4;
}

// [END Messages]


// [START Services]

service Generic {
  rpc TransformInput(SeldonMessage) returns (SeldonMessage) {};
  rpc TransformOutput(SeldonMessage) returns (SeldonMessage) {};
  rpc Route(SeldonMessage) returns (SeldonMessage) {};
  rpc Aggregate(SeldonMessageList) returns (SeldonMessage) {};
  rpc SendFeedback(Feedback) returns (SeldonMessage) {};
}

service Model {
  rpc Predict(SeldonMessage) returns (SeldonMessage) {};
  rpc SendFeedback(Feedback) returns (SeldonMessage) {};
  rpc Metadata(google.protobuf.Empty) returns (SeldonModelMetadata) {};
}

service Router {
  rpc Route(SeldonMessage) returns (SeldonMessage) {};
  rpc SendFeedback(Feedback) returns (SeldonMessage) {};
}

service Transformer {
  rpc TransformInput(SeldonMessage) returns (SeldonMessage) {};
}

service OutputTransformer {
  rpc TransformOutput(SeldonMessage) returns (SeldonMessage) {};
}

service Combiner {
  rpc Aggregate(SeldonMessageList) returns (SeldonMessage) {};
}


service Seldon {
  rpc Predict(SeldonMessage) returns (SeldonMessage) {};
  rpc SendFeedback(Feedback) returns (SeldonMessage) {};
  rpc ModelMetadata(SeldonModelMetadataRequest) returns (SeldonModelMetadata) {};
  rpc GraphMetadata(google.protobuf.Empty) returns (SeldonGraphMetadata) {};
}

// [END Services]
```
