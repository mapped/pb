# PB
This repo contains all the public protobufs for Mapped Platform Service Definitions. Please refer to [Mapped Developer Docs](https://developer.mapped.com/docs) for more information.
 
# Outputs
This monorepo generates packages or outputs for several languages:
- C# - Published as a package to GPR: [Mapped.PB](https://github.com/mapped/pb/packages/1629360)
- Java - Published as a package to GPR: [com.mapped.pb](https://github.com/mapped/pb/packages/)
- TypeScript
  - For Node.JS - Published as a package to GPR: [pb-node](https://github.com/mapped/pb/pkgs/npm/pb-node)
  - For Browsers - Published as a package to GPR: [pb-browser](https://github.com/mapped/pb/pkgs/npm/pb-browser)
- Python - Published on PyPi: [mapped.pb](https://pypi.org/project/mapped.pb/)
- Go - Published as a module to `go.mapped.dev/pb` _(which in turn points to https://github.com/mapped/pb-go)_
  - Use `go get go.mapped.dev/pb/cloud` or `go get go.mapped.dev/pb/gateway`

# GeoJSON Types
All GeoJSON is carried in [`google.protobuf.Struct`](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Struct), as can be seen in [mapped/cloud/types/typed_value.proto](https://github.com/mapped/pb/blob/main/mapped/cloud/types/geojson.proto). Each language's Google Protobuf library has handlers for going from `JSON`<->`google.protobuf.Struct`. To simplify things, this section will list how to do each transform in languages we use at Mapped.

_(All samples skip over normal protobuf message marshal/unmarshal work, as it doesn't change based on the use of structs)_

## C#

From GeoJSON to Struct:
```csharp
string json = File.ReadAllText(@"test.json");
var pbStruct = Google.Protobuf.WellKnownTypes.Struct.Parser.ParseJson(json);
myMsg.Value.GeojsonValue = pbStruct;
```

From Struct to GeoJSON:
```csharp
string json = Google.Protobuf.JsonFormatter.Default.Format(myMsg.Value.GeojsonValue);
```

## Go
_(Samples are missing all error checking for brevity)_

From GeoJSON to Struct:
```go
import (
	"encoding/json"
	"io/ioutil"
	"os"

	"go.mapped.dev/pb/cloud"
	"google.golang.org/protobuf/types/known/structpb"
)

func geojsonToStruct() *cloud.types.TypedValue {
	// Read some json from disk
	jsonFile, _ := os.Open("test.json")
	defer jsonFile.Close()
	byteValue, _ := ioutil.ReadAll(jsonFile)

	// Unmarshal the json
	var jsonMap map[string]interface{}
	json.Unmarshal([]byte(byteValue), &jsonMap)

	// Put the json in to a Struct
	pbStruct, _ := structpb.NewStruct(jsonMap)

	// Use the Struct
	return &cloud.types.TypedValue{
		Value: &cloud.types.TypedValue_GeojsonValue{GeojsonValue: pbStruct},
	}
}
```

From Struct to GeoJSON:
```go
import (
	"encoding/json"
	"io/ioutil"
	"os"

	"go.mapped.dev/pb/cloud"
	"google.golang.org/protobuf/types/known/structpb"
)

func structToGeojson(v *cloud.types.TypedValue) string {
	// Marshal to json
	jsonMap := v.GetGeojsonValue().AsMap()
	jsonBytes, _ := json.Marshal(jsonMap)

	// Return as a string
	return string(jsonBytes)
}
```

# Weirdness

## DO NOT MANUALLY CREATE TAGS
The CI build pipeline creates tags automatically.

