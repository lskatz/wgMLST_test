# wgMLST_test
How much does an MLST database grow per allele?

    # create the first random allele
    perl -e 'print ">locus1_1\n"; @NT=qw(A T C G); for(1..800){print $NT[int(rand(4))];} print "\n";' > ref.fasta
    # Generate a new allele by randomly mutating up to 10 sites.
    # I am being lazy with the ID of the new allele by generating a random number.
    perl -MBio::Perl -e '@NT=qw(A C T G); $seq = Bio::SeqIO->new(-file=>"ref.fasta")->next_seq; $newSeq=$seq->seq; $numMutations=int(rand(9))+1; for(1..$numMutations){my $pos=int(rand($seq->length)); $nt=$NT[int(rand(4))]; $newSeq=substr($newSeq,0,$pos-1).$nt.substr($newSeq,$pos);} $newid = "locus_".int(rand(9999999)); print ">$newid\n$newSeq\n";' >> locus.fasta
    git add ref.fasta
    git commit -m "ref.fasta"

Git was only 538K for me at this point when I ran `du -shc .git`

    # Look at the size per new allele. 
    # Current options are 1000 steps and a step size of 10.
    for i in `seq 1 1000`; do 
      for j in `seq 1 10`; do 
        perl -MBio::Perl -e '
          @NT=qw(A C T G); 
          $seq = Bio::SeqIO->new(-file=>"ref.fasta")->next_seq; 
          $newSeq=$seq->seq; $numMutations=int(rand(9))+1; 
          for(1..$numMutations){
            my $pos=int(rand($seq->length)); 
            $nt=$NT[int(rand(4))]; 
            $newSeq=substr($newSeq,0,$pos-1).$nt.substr($newSeq,$pos);
          } 
          $newid = "locus_".int(rand(9999999)); 
          print ">$newid\n$newSeq\n";
        ' >> locus.fasta; 
        git add locus.fasta;
      done;
      git commit -m "new allele"; 
      du -s .git; 
    done | grep ".git" | tee size.txt

The goodness of fit is very high and so the size requirements are very predictable.  Here is a step size of 100 and number of steps 100 (10,000 total alleles).

[Graph](graph.100.100.png)
