eligibilityDs:Dataset[Eligibility]
medicalDs:Dataset[Medical]
filterMedical:
eligibilityDs:Dataset[Eligibility]
medicalDs:Dataset[Medical]
val semiMedicalDs = medicalDs.alias("m").join(eligibilityDs.alias("e"), col("e.memberID") === col("m.memberID"), "semi")
semiMedicalDs


generatefullName:

medicalDs.withColumn("fullName", concat)



memberId, fullName, paidAmount

val updatedMedicalDs: Dataset[Medical] = medicalDs.alias("m") .join(eligibilityDs.alias("e"), $"e.memberID" === $"m.memberID", "left") .select( col("m.memberId"),memberId concat_ws(" ", col("e.firstName"), col("e.lastName")).as("fullName"),col("m.paidAmount") )

medicalDs.alias("a").join(medicalDs.select(max($"paidAmount").alias("maxPA")).alias("b"), $"a.paidAmount" === $"b.maxPA", "inner").select("memberID").collect()(0).getString(0)
medicalDs.select(sum($"paidAmount")).collect()(0).getLong(0)