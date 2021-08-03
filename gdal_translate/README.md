# gdal_translate

This is a simple example of `gdal_translate` that reads a COG URL and does a geographic subset according to a provided boundinx box and associated EPSG code.

This CWL document takes an URL to a Sentinel-2 STAC asset href pointing to a COG file and uses `gdal_translate` to access the desired portion of the file.

The command wrapped by the CWL `CommandLineTool` is: 

```console
gdal_translate \
    -projwin \
    136.659 \
    -35.791 \
    136.923 \
    -35.96 \
    -projwin_srs \
    EPSG:4326 \
    /vsicurl/https://sentinel-cogs.s3.us-west-2.amazonaws.com/sentinel-s2-l2a-cogs/53/H/PA/2021/7/S2B_53HPA_20210723_0_L2A/B8A.tif \
    B8A.tif
```

The CWL document content is:

```yaml
class: CommandLineTool

requirements:
  InlineJavascriptRequirement: {}
  DockerRequirement: 
    dockerPull: osgeo/gdal

baseCommand: gdal_translate
arguments:
- -projwin 
- valueFrom: ${ return inputs.bbox.split(",")[0]; }
- valueFrom: ${ return inputs.bbox.split(",")[3]; }
- valueFrom: ${ return inputs.bbox.split(",")[2]; }
- valueFrom: ${ return inputs.bbox.split(",")[1]; }
- -projwin_srs
- valueFrom: ${ return inputs.epsg; }
- valueFrom: |
    ${ if (inputs.asset.startsWith("http")) {

         return "/vsicurl/" + inputs.asset; 
       
       } else { 
        
         return inputs.asset;

       } 
    }
- valueFrom: ${ return inputs.asset.split("/").slice(-1)[0]; }

inputs: 
  asset: 
    type: string
  bbox: 
    type: string
  epsg:
    type: string
    default: "EPSG:4326" 

outputs:
  tifs:
    outputBinding:
      glob: '*.tif'
    type: File

cwlVersion: v1.0
```

It may be run with the parameters:

```yaml
bbox: "136.659,-35.96,136.923,-35.791"
asset: https://sentinel-cogs.s3.us-west-2.amazonaws.com/sentinel-s2-l2a-cogs/53/H/PA/2021/7/S2B_53HPA_20210723_0_L2A/B8A.tif
epsg: "EPSG:4326"
```

The execution will generate:

```console
INFO /srv/conda/bin/cwltool 3.0.20210319143721
INFO Resolved 'translate.cwl' to 'file:///home/fbrito/work/sentinel-2-dnbr-multitemporal-cog/translate.cwl'
INFO [job translate.cwl] /tmp/k8idc3ml$ docker \
    run \
    -i \
    --mount=type=bind,source=/tmp/k8idc3ml,target=/MRizBi \
    --mount=type=bind,source=/tmp/zb8r6gau,target=/tmp \
    --workdir=/MRizBi \
    --read-only=true \
    --user=1000:1000 \
    --rm \
    --env=TMPDIR=/tmp \
    --env=HOME=/MRizBi \
    --cidfile=/tmp/57e_gmg_/20210803114536-240117.cid \
    osgeo/gdal \
    gdal_translate \
    -projwin \
    136.659 \
    -35.791 \
    136.923 \
    -35.96 \
    -projwin_srs \
    EPSG:4326 \
    /vsicurl/https://sentinel-cogs.s3.us-west-2.amazonaws.com/sentinel-s2-l2a-cogs/53/H/PA/2021/7/S2B_53HPA_20210723_0_L2A/B8A.tif \
    B8A.tif
Input file size is 5490, 5490
0...10...20...30...40...50...60...70...80...90...100 - done.
INFO [job translate.cwl] Max memory used: 25MiB
INFO [job translate.cwl] completed success
{
    "tifs": {
        "location": "file:///home/fbrito/work/sentinel-2-dnbr-multitemporal-cog/B8A.tif",
        "basename": "B8A.tif",
        "class": "File",
        "checksum": "sha1$033898bb305bb2ae53980182cd882b05cc585fa2",
        "size": 2256036,
        "path": "/home/fbrito/work/sentinel-2-dnbr-multitemporal-cog/B8A.tif"
    }
}
INFO Final process status is success
```