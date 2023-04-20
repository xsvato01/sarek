# Run nf-core Sarek pipeline on K8s

## How to run on from proxy:
  1) Go to folder /mnt/run a create run folder
     <br /> `cd /mnt/run`
     <br /> `mkdir ./RUN_FOLDER`
     <br /> `cd ./RUN_FOLDER` 
 2) Copy nextflow.config and custom.config files, create /tmp dir
     <br /> `cp ../nextflow.config .`
     <br /> `cp ../custom.config .`
     <br /> `mkdir ./tmp`
  3) Edit nextflow.config - launchDir
   
     ```
             k8s {
                 namespace = 'acgt-ns'
                 runAsUser = 1000
                 launchDir = '/mnt/run/RUN_FOLDER'
             }
             executor {
                 queueSize = 400
             }
             process {
                 executor = 'k8s'
                 pod = [ securityContext: [ fsGroupChangePolicy: "OnRootMismatch", runAsUser: 1000, runAsGroup: 1, fsGroup: 1 ] ]

                 withLabel:VEP {
                     memory = {check_resource(30.GB * task.attempt)}
                 }
             }
     ```

  4) Create TSV sample config according to https://nf-co.re/sarek/2.7.1/usage#tsv-file in /mnt/shared/Sarek_configs
  5) Run nextflow (for ACGT germline variant analysis) with our custom Sarek pipeline for https://github.com/Hynst/sarek 
      ```
      nextflow kuberun https://github.com/Hynst/sarek -pod-image 'cerit.io/nextflow:21.09.1' -v 'pvc-acgt:/mnt' \
	  -w /mnt/run/RUN_FOLDER/tmp -c custom.config --input /mnt/shared/Sarek_configs/YOUR_SAMPLE_CONFIG \
	  --genome GRCh38 --skip_markduplicates --tools HaplotypeCaller,VEP,Manta --igenomes_base /mnt/igenome/ --max_time 240.h
      ```
  6) Run nextflow (somatic variant analysis) with our custom Sarek pipeline for https://github.com/Hynst/sarek
     ```
     nextflow kuberun https://github.com/Hynst/sarek -pod-image 'cerit.io/nextflow:21.09.1' -v 'pvc-acgt:/mnt' \
          -w /mnt/run/RUN_FOLDER/tmp -c custom.config --input /mnt/shared/Sarek_configs/YOUR_SAMPLE_CONFIG \
	  --genome GRCh38 --tools FreeBayes,Strelka,Mutect2 --igenomes_base /mnt/igenome/ --max_time 480.h
     ```

## Resume analysis:
   1) Copy again nextflow.config and edit launchDir
   2) Run nextflow command with -resume
      ```
      nextflow kuberun https://github.com/Hynst/sarek -pod-image 'cerit.io/nextflow:21.09.1' -v 'pvc-acgt:/mnt' \
          -w /mnt/run/RUN_FOLDER/tmp -c custom.config --input /mnt/shared/Sarek_configs/YOUR_SAMPLE_CONFIG \
	  --genome GRCh38 --tools FreeBayes,Strelka,Mutect2 --igenomes_base /mnt/igenome/ --max_time 480.h -resume
      ```
