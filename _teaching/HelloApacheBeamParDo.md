---
title: "Apache Beam - Using Pardo"
collection: teaching
type: "Data Processing"
permalink: /teaching/HelloApacheBeamParDo
date: 2024-06-01
venue: "Beam"
date: 2024-06-01
location: "Local"
---

We had performed the same demonstration using SimpleFuntions. Refer [Page](https://nuneskris.github.io/teaching/HelloApacheBeam).  We will be using Pardo Function for this.
Refer https://github.com/nuneskris/hello-apache-beam for detailed description of the demo without Pardo.

```java

public class BeamScoreViaOnlyPardo{
	
	 public static final void main(String args[]) throws Exception {
		 processLocalViaPardo();
	}
    
    private static void processLocalViaPardo(){
        Pipeline pipeline = Pipeline.create();
        pipeline.apply(TextIO.read().from("/Users/krisnunes/eclipse-workspace/HelloApacheBeamPardo/resources/IPL-Ball-by-Ball 2008-2020.csv"))
                .apply(ParDo.of(new BeamScoreViaOnlyPardo.ExtractScore()))
                .apply(ParDo.of(new BeamScoreViaOnlyPardo.FilterWickets()))
                .apply(ParDo.of(new BeamScoreViaOnlyPardo.ConvertToKV()))
                .apply(GroupByKey.<String, Integer>create())
                .apply(ParDo.of(new BeamScoreViaOnlyPardo.SumUpValuesByKey()))
                .apply(TextIO.write().to("/Users/krisnunes/eclipse-workspace/HelloApacheBeamPardo/resources/IPLOutsParDo.csv").withoutSharding())
                ;
        pipeline.run();
    }
    /** A {@link DoFn} that splits lines of text into individual column cells. */
    // The first variable of the doFn is the input and the second variable id the output
    public static class ExtractScore extends DoFn<String, String[]> {
		private static final long serialVersionUID = -4296667754676058140L;
		@ProcessElement
        public void processElement(ProcessContext c) {
            String[] words = c.element().split(",");
            c.output(words);
        }
    }
    // We filter by not calling .output for those rows which are filtered out
    public static class FilterWickets extends  DoFn<String[], String[]> {
		private static final long serialVersionUID = -3042129865531281093L;

		@ProcessElement
        public void processElement(ProcessContext c) {
            if(c.element()[11].equalsIgnoreCase("1")) {
                c.output(c.element());
            }
        }
    }

    public static class ConvertToKV extends  DoFn<String[], KV<String, Integer>> {
		private static final long serialVersionUID = -4110309215590766497L;

		@ProcessElement
        public void processElement(ProcessContext c) {
            String key = c.element()[4] + "," + c.element()[12];
            c.output(KV.of(key, Integer.valueOf(1)));
        }
    }
    public static class SumUpValuesByKey extends DoFn<KV<String, Iterable<Integer>>, String>{
		private static final long serialVersionUID = -7808059852703100891L;

		@ProcessElement
        public void processElement(ProcessContext context) {
            int totalWickets = 0;
            String playerAndWicketType = context.element().getKey();
            Iterable<Integer> wickets = context.element().getValue();
            for (Integer amount : wickets) {
                totalWickets += amount.intValue();
            }
            context.output(playerAndWicketType + "," + totalWickets);
        }
    }
}

```
