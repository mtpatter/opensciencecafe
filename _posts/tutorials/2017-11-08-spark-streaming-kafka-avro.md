---
title: Spark Streaming from Kafka with Avro data
author: maria_patterson
layout: post
permalink: /2017/11/spark-streaming-kafka-avro/
categories:
  - tutorials
tags:
  - python
  - reproducible
  - streaming
  - kafka
  - avro
  - spark

---
Here I show an example of using Spark Streaming to process Avro formatted data from Kafka.

The notebooks for the code below and a Docker image are available here:

[https://github.com/mtpatter/bilao](https://github.com/mtpatter/bilao)

A sample data producer to Kafka used below is available here:

[https://github.com/mtpatter/alert_stream](https://github.com/mtpatter/alert_stream).

You may also need example Avro data here:

[https://github.com/lsst-dm/sample-avro-alert](https://github.com/lsst-dm/sample-avro-alert).


# Spark Streaming from Kafka with Avro formatted data

Example of using Spark to connect to Kafka and using Spark Streaming to process a Kafka stream of Avro (schemaless) objects.

## Set up schemas for decoding

Schemas are pulled automatically from Github during Docker build in mtpatter/alert_stream.


{% highlight python %}
!ls ../../sample-avro-alert/schema
{% endhighlight %}

    alert.avsc   diaobject.avsc  simple.avsc
    cutout.avsc  diasource.avsc  ssobject.avsc



{% highlight python %}
schema_files = ["../../sample-avro-alert/schema/diasource.avsc",
                    "../../sample-avro-alert/schema/diaobject.avsc",
                    "../../sample-avro-alert/schema/ssobject.avsc",
                    "../../sample-avro-alert/schema/cutout.avsc",
                    "../../sample-avro-alert/schema/alert.avsc"]
{% endhighlight %}


{% highlight python %}
import fastavro
import avro.schema
import json


def loadSingleAvsc(file_path, names):
    """Load a single avsc file.
    """
    with open(file_path) as file_text:
        json_data = json.load(file_text)
    schema = avro.schema.SchemaFromJSONData(json_data, names)
    return schema


def combineSchemas(schema_files):
    """Combine multiple nested schemas into a single schema.
    """
    known_schemas = avro.schema.Names()

    for s in schema_files:
        schema = loadSingleAvsc(s, known_schemas)
    return schema.to_json()


schema = combineSchemas(schema_files)


schema
{% endhighlight %}



{% highlight python %}
    {'doc': 'sample avro alert schema v1.0',
     'fields': [{'doc': 'add descriptions like this',
       'name': 'alertId',
       'type': 'long'},
      {'name': 'l1dbId', 'type': 'long'},
      {'name': 'diaSource',
       'type': {'fields': [{'name': 'diaSourceId', 'type': 'long'},
         {'name': 'ccdVisitId', 'type': 'long'},
         {'default': None, 'name': 'diaObjectId', 'type': ['long', 'null']},
         {'default': None, 'name': 'ssObjectId', 'type': ['long', 'null']},
         {'default': None, 'name': 'parentDiaSourceId', 'type': ['long', 'null']},
         {'name': 'midPointTai', 'type': 'double'},
         {'name': 'filterName', 'type': 'string'},
         {'name': 'ra', 'type': 'double'},
         {'name': 'decl', 'type': 'double'},
         {'name': 'ra_decl_Cov',
          'type': [{'fields': [{'name': 'raSigma', 'type': 'float'},
             {'name': 'declSigma', 'type': 'float'},
             {'name': 'ra_decl_Cov', 'type': 'float'}],
            'name': 'ra_decl_Cov',
            'namespace': 'lsst.alert',
            'type': 'record'}]},
         {'name': 'x', 'type': 'float'},
         {'name': 'y', 'type': 'float'},
         {'name': 'x_y_Cov',
          'type': [{'fields': [{'name': 'xSigma', 'type': 'float'},
             {'name': 'ySigma', 'type': 'float'},
             {'name': 'x_y_Cov', 'type': 'float'}],
            'name': 'x_y_Cov',
            'namespace': 'lsst.alert',
            'type': 'record'}]},
         {'default': None,
          'name': 'apFlux',
          'type': [{'fields': [{'aliases': ['base_CircularApertureFlux_3_0_flux'],
              'default': None,
              'name': 'apMeanSb01',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_4_5_flux'],
              'default': None,
              'name': 'apMeanSb02',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_6_0_flux'],
              'default': None,
              'name': 'apMeanSb03',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_9_0_flux'],
              'default': None,
              'name': 'apMeanSb04',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_12_0_flux'],
              'default': None,
              'name': 'apMeanSb05',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_17_0_flux'],
              'default': None,
              'name': 'apMeanSb06',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_25_0_flux'],
              'default': None,
              'name': 'apMeanSb07',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_35_0_flux'],
              'default': None,
              'name': 'apMeanSb08',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_50_0_flux'],
              'default': None,
              'name': 'apMeanSb09',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_70_0_flux'],
              'default': None,
              'name': 'apMeanSb10',
              'type': ['float', 'null']}],
            'name': 'apFlux',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'default': None,
          'name': 'apFluxErr',
          'type': [{'fields': [{'aliases': ['base_CircularApertureFlux_3_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb01Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_4_5_fluxSigma'],
              'default': None,
              'name': 'apMeanSb02Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_6_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb03Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_9_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb04Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_12_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb05Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_17_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb06Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_25_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb07Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_35_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb08Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_50_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb09Err',
              'type': ['float', 'null']},
             {'aliases': ['base_CircularApertureFlux_70_0_fluxSigma'],
              'default': None,
              'name': 'apMeanSb10Err',
              'type': ['float', 'null']}],
            'name': 'apFluxErr',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'name': 'snr', 'type': 'float'},
         {'name': 'psFlux', 'type': 'float'},
         {'default': None, 'name': 'psRa', 'type': ['double', 'null']},
         {'default': None, 'name': 'psDecl', 'type': ['double', 'null']},
         {'default': None,
          'name': 'ps_Cov',
          'type': [{'fields': [{'name': 'psFluxSigma', 'type': 'float'},
             {'name': 'psRaSigma', 'type': 'float'},
             {'name': 'psDeclSigma', 'type': 'float'},
             {'name': 'psFlux_psRa_Cov', 'type': 'float'},
             {'name': 'psFlux_psDecl_Cov', 'type': 'float'},
             {'name': 'psRa_psDecl_Cov', 'type': 'float'}],
            'name': 'ps_Cov',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'default': None, 'name': 'psLnL', 'type': ['float', 'null']},
         {'default': None, 'name': 'psChi2', 'type': ['float', 'null']},
         {'default': None, 'name': 'psNdata', 'type': ['int', 'null']},
         {'default': None, 'name': 'trailFlux', 'type': ['float', 'null']},
         {'default': None, 'name': 'trailRa', 'type': ['double', 'null']},
         {'default': None, 'name': 'trailDecl', 'type': ['double', 'null']},
         {'default': None, 'name': 'trailLength', 'type': ['float', 'null']},
         {'default': None, 'name': 'trailAngle', 'type': ['float', 'null']},
         {'default': None,
          'name': 'trail_Cov',
          'type': [{'fields': [{'name': 'trailFluxSigma', 'type': 'float'},
             {'name': 'trailRaSigma', 'type': 'float'},
             {'name': 'trailDeclSigma', 'type': 'float'},
             {'name': 'trailLengthSigma', 'type': 'float'},
             {'name': 'trailAngleSigma', 'type': 'float'},
             {'name': 'trailFlux_trailRa_Cov', 'type': 'float'},
             {'name': 'trailFlux_trailDecl_Cov', 'type': 'float'},
             {'name': 'trailFlux_trailLength_Cov', 'type': 'float'},
             {'name': 'trailFlux_trailAngle_Cov', 'type': 'float'},
             {'name': 'trailRa_trailDecl_Cov', 'type': 'float'},
             {'name': 'trailRa_trailLength_Cov', 'type': 'float'},
             {'name': 'trailRa_trailAngle_Cov', 'type': 'float'},
             {'name': 'trailDecl_trailLength_Cov', 'type': 'float'},
             {'name': 'trailDecl_trailAngle_Cov', 'type': 'float'},
             {'name': 'trailLength_trailAngle_Cov', 'type': 'float'}],
            'name': 'trail_Cov',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'default': None, 'name': 'trailLnL', 'type': ['float', 'null']},
         {'default': None, 'name': 'trailChi2', 'type': ['float', 'null']},
         {'default': None, 'name': 'trailNdata', 'type': ['int', 'null']},
         {'default': None, 'name': 'dipMeanFlux', 'type': ['float', 'null']},
         {'default': None, 'name': 'dipFluxDiff', 'type': ['float', 'null']},
         {'default': None, 'name': 'dipRa', 'type': ['double', 'null']},
         {'default': None, 'name': 'dipDecl', 'type': ['double', 'null']},
         {'default': None, 'name': 'dipLength', 'type': ['float', 'null']},
         {'default': None, 'name': 'dipAngle', 'type': ['float', 'null']},
         {'default': None,
          'name': 'dip_Cov',
          'type': [{'fields': [{'name': 'dipMeanFluxSigma', 'type': 'float'},
             {'name': 'dipFluxDiffSigma', 'type': 'float'},
             {'name': 'dipRaSigma', 'type': 'float'},
             {'name': 'dipDeclSigma', 'type': 'float'},
             {'name': 'dipLengthSigma', 'type': 'float'},
             {'name': 'dipAngleSigma', 'type': 'float'},
             {'name': 'dipMeanFlux_dipFluxDiff_Cov', 'type': 'float'},
             {'name': 'dipMeanFlux_dipRa_Cov', 'type': 'float'},
             {'name': 'dipMeanFlux_dipDecl_Cov', 'type': 'float'},
             {'name': 'dipMeanFlux_dipLength_Cov', 'type': 'float'},
             {'name': 'dipMeanFlux_dipAngle_Cov', 'type': 'float'},
             {'name': 'dipFluxDiff_dipRa_Cov', 'type': 'float'},
             {'name': 'dipFluxDiff_dipDecl_Cov', 'type': 'float'},
             {'name': 'dipFluxDiff_dipLength_Cov', 'type': 'float'},
             {'name': 'dipFluxDiff_dipAngle_Cov', 'type': 'float'},
             {'name': 'dipRa_dipDecl_Cov', 'type': 'float'},
             {'name': 'dipRa_dipLength_Cov', 'type': 'float'},
             {'name': 'dipRa_dipAngle_Cov', 'type': 'float'},
             {'name': 'dipDecl_dipLength_Cov', 'type': 'float'},
             {'name': 'dipDecl_dipAngle_Cov', 'type': 'float'},
             {'name': 'dipLength_dipAngle_Cov', 'type': 'float'}],
            'name': 'dip_Cov',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'default': None, 'name': 'dipLnL', 'type': ['float', 'null']},
         {'default': None, 'name': 'dipChi2', 'type': ['float', 'null']},
         {'default': None, 'name': 'dipNdata', 'type': ['int', 'null']},
         {'aliases': ['fpFlux'],
          'default': None,
          'name': 'totFlux',
          'type': ['float', 'null']},
         {'aliases': ['fpFluxSigma'],
          'default': None,
          'name': 'totFluxErr',
          'type': ['float', 'null']},
         {'default': None, 'name': 'diffFlux', 'type': ['float', 'null']},
         {'default': None, 'name': 'diffFluxErr', 'type': ['float', 'null']},
         {'aliases': ['fpSky'],
          'default': None,
          'name': 'fpBkgd',
          'type': ['float', 'null']},
         {'aliases': ['fpSkySigma'],
          'default': None,
          'name': 'fpBkgdErr',
          'type': ['float', 'null']},
         {'default': None, 'name': 'ixx', 'type': ['float', 'null']},
         {'default': None, 'name': 'iyy', 'type': ['float', 'null']},
         {'default': None, 'name': 'ixy', 'type': ['float', 'null']},
         {'default': None,
          'name': 'i_cov',
          'type': [{'fields': [{'name': 'ixxSigma', 'type': 'float'},
             {'name': 'iyySigma', 'type': 'float'},
             {'name': 'ixySigma', 'type': 'float'},
             {'name': 'ixx_iyy_Cov', 'type': 'float'},
             {'name': 'ixx_ixy_Cov', 'type': 'float'},
             {'name': 'iyy_ixy_Cov', 'type': 'float'}],
            'name': 'i_cov',
            'namespace': 'lsst.alert',
            'type': 'record'},
           'null']},
         {'default': None, 'name': 'ixxPSF', 'type': ['float', 'null']},
         {'default': None, 'name': 'iyyPSF', 'type': ['float', 'null']},
         {'default': None, 'name': 'ixyPSF', 'type': ['float', 'null']},
         {'default': None, 'name': 'extendedness', 'type': ['float', 'null']},
         {'default': None, 'name': 'spuriousness', 'type': ['float', 'null']},
         {'name': 'flags', 'type': 'long'}],
        'name': 'diaSource',
        'namespace': 'lsst.alert',
        'type': 'record'}},
      {'default': None,
       'name': 'prv_diaSources',
       'type': [{'items': 'lsst.alert.diaSource', 'type': 'array'}, 'null']},
      {'default': None,
       'name': 'diaObject',
       'type': [{'fields': [{'name': 'diaObjectId', 'type': 'long'},
          {'name': 'ra', 'type': 'double'},
          {'name': 'decl', 'type': 'double'},
          {'name': 'ra_decl_Cov',
           'type': [{'fields': [{'name': 'raSigma', 'type': 'float'},
              {'name': 'declSigma', 'type': 'float'},
              {'name': 'ra_decl_Cov', 'type': 'float'}],
             'name': 'ra_decl_Cov',
             'namespace': 'lsst',
             'type': 'record'}]},
          {'name': 'radecTai', 'type': 'double'},
          {'name': 'pmRa', 'type': 'float'},
          {'name': 'pmDecl', 'type': 'float'},
          {'name': 'parallax', 'type': 'float'},
          {'name': 'pm_parallax_Cov',
           'type': [{'fields': [{'name': 'pmRaSigma', 'type': 'float'},
              {'name': 'pmDeclSigma', 'type': 'float'},
              {'name': 'parallaxSigma', 'type': 'float'},
              {'name': 'pmRa_pmDecl_Cov', 'type': 'float'},
              {'name': 'pmRa_parallax_Cov', 'type': 'float'},
              {'name': 'pmDecl_parallax_Cov', 'type': 'float'}],
             'name': 'pm_parallax_Cov',
             'namespace': 'lsst',
             'type': 'record'}]},
          {'name': 'pmParallaxLnL', 'type': 'float'},
          {'name': 'pmParallaxChi2', 'type': 'float'},
          {'name': 'pmParallaxNdata', 'type': 'int'},
          {'default': None, 'name': 'uPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'uPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'uPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'uPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'uPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'gPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'gPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'gPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'gPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'gPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'rPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'rPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'rPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'rPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'rPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'iPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'iPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'iPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'iPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'iPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'zPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'zPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'zPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'zPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'zPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'yPSFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'yPSFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'yPSFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'yPSFluxChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'yPSFluxNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'uFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'uFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'uFPFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'gFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'gFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'gFPFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'rFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'rFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'rFPFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'iFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'iFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'iFPFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'zFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'zFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'zFPFluxSigma', 'type': ['float', 'null']},
          {'default': None, 'name': 'yFPFluxMean', 'type': ['float', 'null']},
          {'default': None, 'name': 'yFPFluxMeanErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'yFPFluxSigma', 'type': ['float', 'null']},
          {'default': None,
           'name': 'uLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'uLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'uLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'gLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'gLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'gLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'rLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'rLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'rLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'iLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'iLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'iLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'zLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'zLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'zLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'yLcPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'yLcPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic20',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic21',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic22',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic23',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic24',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic25',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic26',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic27',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic28',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic29',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic30',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic31',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcPeriodic32',
               'type': ['float', 'null']}],
             'name': 'yLcPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'uLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'uLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'uLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'uLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'gLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'gLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'gLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'gLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'rLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'rLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'rLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'rLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'iLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'iLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'iLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'iLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'zLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'zLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'zLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'zLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'yLcNonPeriodic',
           'type': [{'fields': [{'default': None,
               'name': 'yLcNonPeriodic01',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic02',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic03',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic04',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic05',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic06',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic07',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic08',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic09',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic10',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic11',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic12',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic13',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic14',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic15',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic16',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic17',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic18',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic19',
               'type': ['float', 'null']},
              {'default': None,
               'name': 'yLcNonPeriodic20',
               'type': ['float', 'null']}],
             'name': 'yLcNonPeriodic',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None, 'name': 'nearbyObj1', 'type': ['long', 'null']},
          {'default': None, 'name': 'nearbyObj1Dist', 'type': ['float', 'null']},
          {'default': None, 'name': 'nearbyObj1LnP', 'type': ['float', 'null']},
          {'default': None, 'name': 'nearbyObj2', 'type': ['long', 'null']},
          {'default': None, 'name': 'nearbyObj2Dist', 'type': ['float', 'null']},
          {'default': None, 'name': 'nearbyObj2LnP', 'type': ['float', 'null']},
          {'default': None, 'name': 'nearbyObj3', 'type': ['long', 'null']},
          {'default': None, 'name': 'nearbyObj3Dist', 'type': ['float', 'null']},
          {'default': None, 'name': 'nearbyObj3LnP', 'type': ['float', 'null']},
          {'name': 'flags', 'type': 'long'}],
         'name': 'diaObject',
         'namespace': 'lsst',
         'type': 'record'},
        'null']},
      {'default': None,
       'name': 'ssObject',
       'type': [{'fields': [{'name': 'ssObjectId', 'type': 'long'},
          {'default': None,
           'name': 'oe',
           'type': [{'fields': [{'default': None,
               'name': 'q',
               'type': ['double', 'null']},
              {'default': None, 'name': 'e', 'type': ['double', 'null']},
              {'default': None, 'name': 'i', 'type': ['double', 'null']},
              {'default': None, 'name': 'lan', 'type': ['double', 'null']},
              {'default': None, 'name': 'aop', 'type': ['double', 'null']},
              {'default': None, 'name': 'M', 'type': ['double', 'null']},
              {'default': None, 'name': 'epoch', 'type': ['double', 'null']}],
             'name': 'oe',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None,
           'name': 'oe_Cov',
           'type': [{'fields': [{'default': None,
               'name': 'qSigma',
               'type': ['double', 'null']},
              {'default': None, 'name': 'eSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'iSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'lanSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'aopSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'MSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'epochSigma', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_e_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_i_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_lan_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_aop_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_M_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'q_epoch_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'e_i_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'e_lan_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'e_aop_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'e_M_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'e_epoch_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'i_lan_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'i_aop_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'i_M_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'i_epoch_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'lan_aop_Cov', 'type': ['double', 'null']},
              {'default': None, 'name': 'lan_M_Cov', 'type': ['double', 'null']},
              {'default': None,
               'name': 'lan_epoch_Cov',
               'type': ['double', 'null']},
              {'default': None, 'name': 'aop_M_Cov', 'type': ['double', 'null']},
              {'default': None,
               'name': 'aop_epoch_Cov',
               'type': ['double', 'null']},
              {'default': None,
               'name': 'M_epoch_Cov',
               'type': ['double', 'null']}],
             'name': 'oe_Cov',
             'namespace': 'lsst',
             'type': 'record'},
            'null']},
          {'default': None, 'name': 'arc', 'type': ['float', 'null']},
          {'default': None, 'name': 'orbFitLnL', 'type': ['float', 'null']},
          {'default': None, 'name': 'orbFitChi2', 'type': ['float', 'null']},
          {'default': None, 'name': 'orbFitNdata', 'type': ['int', 'null']},
          {'default': None, 'name': 'MOID1', 'type': ['float', 'null']},
          {'default': None, 'name': 'MOID2', 'type': ['float', 'null']},
          {'default': None, 'name': 'moidLon1', 'type': ['double', 'null']},
          {'default': None, 'name': 'moidLon2', 'type': ['double', 'null']},
          {'default': None, 'name': 'uH', 'type': ['float', 'null']},
          {'default': None, 'name': 'uHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'uG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'uG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'uG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'uG2Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'gH', 'type': ['float', 'null']},
          {'default': None, 'name': 'gHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'gG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'gG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'gG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'gG2Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'rH', 'type': ['float', 'null']},
          {'default': None, 'name': 'rHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'rG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'rG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'rG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'rG2Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'iH', 'type': ['float', 'null']},
          {'default': None, 'name': 'iHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'iG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'iG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'iG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'iG2Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'zH', 'type': ['float', 'null']},
          {'default': None, 'name': 'zHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'zG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'zG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'zG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'zG2Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'yH', 'type': ['float', 'null']},
          {'default': None, 'name': 'yHErr', 'type': ['float', 'null']},
          {'default': None, 'name': 'yG1', 'type': ['float', 'null']},
          {'default': None, 'name': 'yG1Err', 'type': ['float', 'null']},
          {'default': None, 'name': 'yG2', 'type': ['float', 'null']},
          {'default': None, 'name': 'yG2Err', 'type': ['float', 'null']},
          {'name': 'flags', 'type': 'long'}],
         'name': 'ssObject',
         'namespace': 'lsst',
         'type': 'record'},
        'null']},
      {'default': None, 'name': 'diaObjectL2', 'type': ['lsst.diaObject', 'null']},
      {'default': None,
       'name': 'diaSourcesL2',
       'type': [{'items': 'lsst.alert.diaSource', 'type': 'array'}, 'null']},
      {'default': None,
       'name': 'cutoutDifference',
       'type': [{'fields': [{'name': 'fileName', 'type': 'string'},
          {'name': 'stampData', 'type': 'bytes'}],
         'name': 'cutout',
         'namespace': 'lsst.alert',
         'type': 'record'},
        'null']},
      {'default': None,
       'name': 'cutoutTemplate',
       'type': ['lsst.alert.cutout', 'null']}],
     'name': 'alert',
     'namespace': 'lsst',
     'type': 'record'}
{% endhighlight %}



## Prep Spark environment

Need some packages to talk to Kafka.


{% highlight python %}
import os
os.environ['PYSPARK_SUBMIT_ARGS'] = '--packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.1.0,org.apache.spark:spark-sql-kafka-0-10_2.11:2.1.0,com.databricks:spark-avro_2.11:3.2.0 pyspark-shell'
{% endhighlight %}

## Create streaming context

To do stream processing with Spark (which uses micro-batches), you need to set a "streaming context" with a time-based window, which sets the environment for executing processes on the stream (as mini-batches). Below, I initiate the streaming context to a batch size of 10 seconds. What happens then is that Spark takes the defined filters/ maps /etc that operate on the stream and executes them at the interval defined by the streaming context, resulting in a new stream. I'm printing that output stream to screen here. At the regularly defined interval (10 seconds), the output of the filters is printed. Given the input of the 10 alert bursts at every ~39 seconds, the output will be printed to match (sometimes 0 and sometimes it will be 10 and the dummy filter that should catch no alerts should always be 0).


{% highlight python %}
from pyspark import SparkContext
from pyspark.streaming import StreamingContext
from pyspark.streaming.kafka import KafkaUtils

# create spark and streaming contexts
sc = SparkContext("local[*]", "KafkaDirectStreamAvro")
ssc = StreamingContext(sc, 10)

# defining the checkpoint directory
ssc.checkpoint("/tmp")
sc.setLogLevel("ERROR")

{% endhighlight %}

## Start a Kafka stream for Spark to subscribe

No stamps, uses Avro schemaless encoding.

With mtpatter/alert_stream, in an external shell:

{% highlight python %}
docker run -it       --network=alertstream_default       alert_stream python bin/sendAlertStream.py my-stream 10 --no-stamps --repeat --max-repeats 3
{% endhighlight %}


## Create output for Spark to print

kafkaStream is configured with decoding applied to the value directly.

alerts grabs the actual alert messages.

alertIds applies a map that just grabs individual alertId's.

filter_all demonstrates a filtered stream that should catch all the alerts.

filter_empty demonstrates a filtered stream that should be empty.


{% highlight python %}
import io

def decoder(msg):
    bytes_io = io.BytesIO(msg)
    bytes_io.seek(0)
    alert = fastavro.schemaless_reader(bytes_io, schema)
    return alert


kafkaStream = KafkaUtils.createDirectStream(ssc, ['my-stream'], {'bootstrap.servers': 'kafka:9092',
            'auto.offset.reset': 'smallest', 'group.id': 'spark-group' }, valueDecoder=decoder)

alerts = kafkaStream.map(lambda x: x[1])
alerts.pprint()


def map_alertId(alert):
    return alert['alertId']


alertIds = alerts.map(map_alertId)
alertIds.count().map(lambda x:'AlertId alerts in this window: %s' % x).pprint()  
alertIds.pprint()


def filter_allRa(alert):
    return alert['diaSource']['ra'] > 350


filter_all = alerts.filter(filter_allRa)
filter_all.count().map(lambda x:'Filter_all alerts in this window: %s' % x).pprint()  
filter_all.pprint()


def filter_emptyRa(alert):
    return alert['diaSource']['ra'] < 350


filter_empty = alerts.filter(filter_emptyRa)
filter_empty.count().map(lambda x:'Filter_empty alerts in this window: %s' % x).pprint()  
filter_empty.pprint()
{% endhighlight %}

## Start the streaming context

Output pprints of the streams above appear.


{% highlight python %}
ssc.start()
ssc.awaitTermination()

    -------------------------------------------
    Time: 2017-11-08 22:14:20
    -------------------------------------------
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': {'fileName': 'stamp-676.fits', 'stampData': b'SIMPLE  =                    T / conforms to FITS standard                      BITPIX  =                  -32 / array data type                                NAXIS   =                    2 / number of array dimensions                     NAXIS1  =                   31                                                  NAXIS2  =                   31                                                  EXTEND  =                    T                                                  CD1_1   = -3.0298457783144E-05                                                  CD1_2   = 4.65770542146534E-05                                                  CD2_1   = -4.6588044497537E-05                                                  CD2_2   = -3.0306475118993E-05                                                  CRPIX1  =    1929.722072056903                                                  CRPIX1A =                    1                                                  CRPIX2  =    214.8792590683224                                                  CRPIX2A =                    1                                                  CRVAL1  =    53.00566810681171                                                  CRVAL1A =                  122                                                  CRVAL2  =   -27.44262443999945                                                  CRVAL2A =                 1885                                                  CTYPE1  = \'RA---TAN-SIP\'                                                        CTYPE1A = \'LINEAR  \'                                                            CTYPE2  = \'DEC--TAN-SIP\'                                                        CTYPE2A = \'LINEAR  \'                                                            CUNIT1  = \'deg     \'                                                            CUNIT1A = \'PIXEL   \'                                                            CUNIT2  = \'deg     \'                                                            CUNIT2A = \'PIXEL   \'                                                            EQUINOX =               2000.0                                                  FILTER  = \'r       \'                                                            FLUXMAG0=     6575938845436.84                                                  HIERARCH FLUXMAG0ERR = 1340273802.27781                                         LTV1    =                 -122                                                  LTV2    =                -1885                                                  RADESYS = \'FK5     \'                                                            EXTTYPE = \'IMAGE   \'                                                            EXTNAME = \'IMAGE   \'                                                            END                                                   
    -------------------------------------------
    Time: 2017-11-08 22:14:20
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:20
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------
    AlertId alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------
    Filter_all alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:30
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------
    AlertId alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------
    Filter_all alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:40
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------
    AlertId alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------
    Filter_all alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:14:50
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    AlertId alerts in this window: 10

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321
    1231321321

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    Filter_all alerts in this window: 10

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}
    {'alertId': 1231321321, 'l1dbId': 222222222, 'diaSource': {'diaSourceId': 281323062375219200, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, 'prv_diaSources': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'diaObject': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'ssObject': {'ssObjectId': 5364546, 'oe': {'q': 6654.14, 'e': 636.121, 'i': 5436.2344, 'lan': 54325.34, 'aop': 344243.3, 'M': 131.1241, 'epoch': 134141.0}, 'oe_Cov': None, 'arc': 2.124124050140381, 'orbFitLnL': 1343141.0, 'orbFitChi2': 1341421.25, 'orbFitNdata': 1214, 'MOID1': 3141.0, 'MOID2': 23432.423828125, 'moidLon1': 2143.213, 'moidLon2': 3142.23123, 'uH': 13231.2314453125, 'uHErr': 13213.212890625, 'uG1': 32131.3125, 'uG1Err': 31232.212890625, 'uG2': 231.23129272460938, 'uG2Err': 23132.23046875, 'gH': None, 'gHErr': None, 'gG1': None, 'gG1Err': None, 'gG2': None, 'gG2Err': None, 'rH': None, 'rHErr': None, 'rG1': None, 'rG1Err': None, 'rG2': None, 'rG2Err': None, 'iH': None, 'iHErr': None, 'iG1': None, 'iG1Err': None, 'iG2': None, 'iG2Err': None, 'zH': None, 'zHErr': None, 'zG1': None, 'zG1Err': None, 'zG2': None, 'zG2Err': None, 'yH': None, 'yHErr': None, 'yG1': None, 'yG1Err': None, 'yG2': None, 'yG2Err': None, 'flags': 0}, 'diaObjectL2': {'diaObjectId': 281323062375219201, 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'radecTai': 1480360995.0, 'pmRa': 0.00013000000035390258, 'pmDecl': 0.00014000000373926014, 'parallax': 2.124124050140381, 'pm_parallax_Cov': {'pmRaSigma': 0.00013000000035390258, 'pmDeclSigma': 0.00013000000035390258, 'parallaxSigma': 0.00013000000035390258, 'pmRa_pmDecl_Cov': 0.00013000000035390258, 'pmRa_parallax_Cov': 0.00013000000035390258, 'pmDecl_parallax_Cov': 0.00013000000035390258}, 'pmParallaxLnL': 0.00013000000035390258, 'pmParallaxChi2': 0.00013000000035390258, 'pmParallaxNdata': 1214, 'uPSFluxMean': None, 'uPSFluxMeanErr': None, 'uPSFluxSigma': None, 'uPSFluxChi2': None, 'uPSFluxNdata': None, 'gPSFluxMean': None, 'gPSFluxMeanErr': None, 'gPSFluxSigma': None, 'gPSFluxChi2': None, 'gPSFluxNdata': None, 'rPSFluxMean': None, 'rPSFluxMeanErr': None, 'rPSFluxSigma': None, 'rPSFluxChi2': None, 'rPSFluxNdata': None, 'iPSFluxMean': None, 'iPSFluxMeanErr': None, 'iPSFluxSigma': None, 'iPSFluxChi2': None, 'iPSFluxNdata': None, 'zPSFluxMean': None, 'zPSFluxMeanErr': None, 'zPSFluxSigma': None, 'zPSFluxChi2': None, 'zPSFluxNdata': None, 'yPSFluxMean': None, 'yPSFluxMeanErr': None, 'yPSFluxSigma': None, 'yPSFluxChi2': None, 'yPSFluxNdata': None, 'uFPFluxMean': None, 'uFPFluxMeanErr': None, 'uFPFluxSigma': None, 'gFPFluxMean': None, 'gFPFluxMeanErr': None, 'gFPFluxSigma': None, 'rFPFluxMean': None, 'rFPFluxMeanErr': None, 'rFPFluxSigma': None, 'iFPFluxMean': None, 'iFPFluxMeanErr': None, 'iFPFluxSigma': None, 'zFPFluxMean': None, 'zFPFluxMeanErr': None, 'zFPFluxSigma': None, 'yFPFluxMean': None, 'yFPFluxMeanErr': None, 'yFPFluxSigma': None, 'uLcPeriodic': None, 'gLcPeriodic': None, 'rLcPeriodic': None, 'iLcPeriodic': None, 'zLcPeriodic': None, 'yLcPeriodic': None, 'uLcNonPeriodic': None, 'gLcNonPeriodic': None, 'rLcNonPeriodic': None, 'iLcNonPeriodic': None, 'zLcNonPeriodic': None, 'yLcNonPeriodic': None, 'nearbyObj1': None, 'nearbyObj1Dist': None, 'nearbyObj1LnP': None, 'nearbyObj2': None, 'nearbyObj2Dist': None, 'nearbyObj2LnP': None, 'nearbyObj3': None, 'nearbyObj3Dist': None, 'nearbyObj3LnP': None, 'flags': 0}, 'diaSourcesL2': [{'diaSourceId': 281323062375219198, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}, {'diaSourceId': 281323062375219199, 'ccdVisitId': 111111, 'diaObjectId': None, 'ssObjectId': None, 'parentDiaSourceId': None, 'midPointTai': 1480360995.0, 'filterName': 'r', 'ra': 351.570546978, 'decl': 0.126243049656, 'ra_decl_Cov': {'raSigma': 0.0002800000074785203, 'declSigma': 0.0002800000074785203, 'ra_decl_Cov': 0.0002899999963119626}, 'x': 112.0999984741211, 'y': 121.0999984741211, 'x_y_Cov': {'xSigma': 1.2000000476837158, 'ySigma': 1.100000023841858, 'x_y_Cov': 1.2000000476837158}, 'apFlux': None, 'apFluxErr': None, 'snr': 41.099998474121094, 'psFlux': 1241.0, 'psRa': None, 'psDecl': None, 'ps_Cov': None, 'psLnL': None, 'psChi2': None, 'psNdata': None, 'trailFlux': None, 'trailRa': None, 'trailDecl': None, 'trailLength': None, 'trailAngle': None, 'trail_Cov': None, 'trailLnL': None, 'trailChi2': None, 'trailNdata': None, 'dipMeanFlux': None, 'dipFluxDiff': None, 'dipRa': None, 'dipDecl': None, 'dipLength': None, 'dipAngle': None, 'dip_Cov': None, 'dipLnL': None, 'dipChi2': None, 'dipNdata': None, 'totFlux': None, 'totFluxErr': None, 'diffFlux': None, 'diffFluxErr': None, 'fpBkgd': None, 'fpBkgdErr': None, 'ixx': None, 'iyy': None, 'ixy': None, 'i_cov': None, 'ixxPSF': None, 'iyyPSF': None, 'ixyPSF': None, 'extendedness': None, 'spuriousness': None, 'flags': 0}], 'cutoutDifference': None, 'cutoutTemplate': None}

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:15:00
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------
    AlertId alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------
    Filter_all alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------
    Filter_empty alerts in this window: 0

    -------------------------------------------
    Time: 2017-11-08 22:15:10
    -------------------------------------------




    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-18-d970cfd6a21f> in <module>()
          1 ssc.start()
    ----> 2 ssc.awaitTermination()


    /usr/local/spark/python/pyspark/streaming/context.py in awaitTermination(self, timeout)
        204         """
        205         if timeout is None:
    --> 206             self._jssc.awaitTermination()
        207         else:
        208             self._jssc.awaitTerminationOrTimeout(int(timeout * 1000))


    /usr/local/spark/python/lib/py4j-0.10.4-src.zip/py4j/java_gateway.py in __call__(self, *args)
       1129             proto.END_COMMAND_PART
       1130
    -> 1131         answer = self.gateway_client.send_command(command)
       1132         return_value = get_return_value(
       1133             answer, self.gateway_client, self.target_id, self.name)


    /usr/local/spark/python/lib/py4j-0.10.4-src.zip/py4j/java_gateway.py in send_command(self, command, retry, binary)
        881         connection = self._get_connection()
        882         try:
    --> 883             response = connection.send_command(command)
        884             if binary:
        885                 return response, self._create_connection_guard(connection)


    /usr/local/spark/python/lib/py4j-0.10.4-src.zip/py4j/java_gateway.py in send_command(self, command)
       1026
       1027         try:
    -> 1028             answer = smart_decode(self.stream.readline()[:-1])
       1029             logger.debug("Answer received: {0}".format(answer))
       1030             if answer.startswith(proto.RETURN_MESSAGE):


    /opt/conda/lib/python3.6/socket.py in readinto(self, b)
        584         while True:
        585             try:
    --> 586                 return self._sock.recv_into(b)
        587             except timeout:
        588                 self._timeout_occurred = True


    KeyboardInterrupt:



ssc.stop()
sc.stop()
{% endhighlight %}
