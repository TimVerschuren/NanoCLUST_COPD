# NanoCLUST_COPD

This GitHub repository consists of two altered files utilised by NanoCLUST. Replace the main.nf and nextflow.config files in the original NanoCLUST fileset with these new files. The main.nf script has a stricter classification algorithm through the addition of a higher e-value requirement (1e-40) and a minimum percentage identiy (99). If an altered evalue or minimum percentage identity is required, change the "evalue" value and the "perc_identity" value in the main.nf script. 
Look for the following piece of code:
==========================================================================================================================================================
 process consensus_classification {
     publishDir "${params.outdir}/${barcode}/cluster${cluster_id}", mode: 'copy', pattern: 'consensus_classification.csv'
     time '3m'
     errorStrategy { sleep(1000); return 'retry' }
     maxRetries 5

     input:
     tuple val(barcode), val(cluster_id), file(cluster_log), file(consensus) from final_consensus

     output:
     file('consensus_classification.csv')
     tuple val(barcode), file('*_blast.log') into classifications_ch

     script:
     db = resolve_blast_db_path(params.db)
     taxdb = resolve_blast_db_path(params.tax)

     if(!params.db)
        """
        blastn -query $consensus -db nr -remote -entrez_query "Bacteria [Organism]" -task blastn -dust no -outfmt "10 staxids sscinames evalue length score pident" -evalue 1E-40 -max_hsps 50 -max_target_seqs 5 -perc_identity 99> consensus_classification.csv
        cat $cluster_log > ${cluster_id}_blast.log
        echo -n ";" >> ${cluster_id}_blast.log
        BLAST_OUT=\$(cut -d";" -f1,2,4,5 consensus_classification.csv | head -n1)
        echo \$BLAST_OUT >> ${cluster_id}_blast.log
        """

    else
        """
        export BLASTDB=
        export BLASTDB=\$BLASTDB:$taxdb
        blastn -query $consensus -db $db -task blastn -dust no -outfmt "10 sscinames staxids evalue length pident" -evalue 1E-40 -max_hsps 50 -max_target_seqs 5 perc_identity 99| sed 's/,/;/g' > consensus_classification.csv
        #DECIDE FINAL CLASSIFFICATION
        cat $cluster_log > ${cluster_id}_blast.log
        echo -n ";" >> ${cluster_id}_blast.log
        BLAST_OUT=\$(cut -d";" -f1,2,4,5 consensus_classification.csv | head -n1)
        echo \$BLAST_OUT >> ${cluster_id}_blast.log
        """
 }
 ==========================================================================================================================================================
