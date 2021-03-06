# Configuring Encryption Options<a name="encryption"></a>

You can use Athena to query encrypted data in Amazon S3 by indicating data encryption when you create a table\. You can also choose to encrypt the results of all queries in Amazon S3, which Athena stores in a location known as the *S3 staging directory*\. You can encrypt query results stored in Amazon S3 whether the underlying dataset is encrypted in Amazon S3 or not\. You set up query\-result encryption using the Athena console or, if you connect using the JDBC driver, by configuring driver options\. You specify the type of encryption to use and the Amazon S3 staging directory location\. Query\-result encryption applies to all queries\.

These options encrypt data at rest in Amazon S3\. Regardless of whether you use these options, transport layer security \(TLS\) encrypts objects in\-transit between Athena resources and between Athena and Amazon S3\. Query results stream to JDBC clients as plain text and are encrypted using SSL\.

**Important**  
The setup for querying an encrypted dataset in Amazon S3 and the options in Athena to encrypt query results are independent\. Each option is enabled and configured separately\. You can use different encryption methods or keys for each\. This means that reading encrypted data in Amazon S3 doesn't automatically encrypt Athena query results in Amazon S3\. The opposite is also true\. Encrypting Athena query results in Amazon S3 doesn't encrypt the underlying dataset in Amazon S3\.

Athena supports the following S3 encryption options, both for encrypted datasets in Amazon S3 and for encrypted query results:

+ Server side encryption with an Amazon S3\-managed key \([SSE\-S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html)\)

+ Server\-side encryption with a AWS KMS\-managed key \([SSE\-KMS](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)\)\.
**Note**  
With SSE\-KMS, Athena does not require you to indicate data is encrypted when creating a table\.

+ Client\-side encryption with a AWS KMS\-managed key \([CSE\-KMS](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html#client-side-encryption-kms-managed-master-key-intro)\)

For more information about AWS KMS encryption with Amazon S3, see [What is AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/overview.html) and [How Amazon Simple Storage Service \(Amazon S3\) Uses AWS KMS](http://docs.aws.amazon.com/kms/latest/developerguide/services-s3.html) in the *AWS Key Management Service Developer Guide*\.

Athena does not support SSE with customer\-provided keys \(SSE\-C\), nor does it support client\-side encryption using a client\-side master key\. To compare Amazon S3 encryption options, see [Protecting Data Using Encryption](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html) in the *Amazon Simple Storage Service Developer Guide*\.

Athena does not support running queries from one region on encrypted data stored in Amazon S3 in another region\.

## Permissions for Encrypting and Decrypting Data<a name="permissions-for-encrypting-and-decrypting-data"></a>

If you use SSE\-S3 for encryption, Athena users require no additional permissions for encryption and decryption\. Having the appropriate Amazon S3 permissions for the appropriate Amazon S3 location \(and for Athena actions\) is enough\. For more information about policies that allow appropriate Athena and Amazon S3 permissions, see [IAM Policies for User Access](access.md#managed-policies) and [Amazon S3 Permissions](access.md#s3-permissions)\.

For data that is encrypted using AWS KMS, Athena users must be allowed to perform particular AWS KMS actions in addition to Athena and S3 permissions\. You allow these actions by editing the key policy for the KMS customer master keys \(CMKs\) that are used to encrypt data in Amazon S3\. The easiest way to do this is to use the IAM console to add key users to the appropriate KMS key policies\. For information about how to add a user to a KMS key policy, see [How to Modify a Key Policy](http://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying.html#key-policy-modifying-how-to-console-default-view) in the *AWS Key Management Service Developer Guide*\.

**Note**  
Advanced key policy administrators may want to fine\-tune key policies\. `kms:Decrypt` is the minimum allowed action for an Athena user to work with an encrypted dataset\. To work with encrypted query results, the minimum allowed actions are `kms:GenerateDataKey` and `kms:Decrypt`\.

When using Athena to query datasets in Amazon S3 with a large number of objects that are encrypted with AWS KMS, AWS KMS may throttle query results\. This is more likely when there are a large number of small objects\. Athena backs off retry requests, but a throttling error might still occur\. In this case, visit the [AWS Support Center](https://console.aws.amazon.com/support/home) and create a case to increase your limit\. For more information about limits and AWS KMS throttling, see [Limits](http://docs.aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second) in the *AWS Key Management Service Developer Guide*\.

## Creating Tables Based on Encrypted Datasets in Amazon S3<a name="creating-tables-based-on-encrypted-datasets-in-s3"></a>

You indicate to Athena that a dataset is encrypted in Amazon S3 when you create a table \(this is not required when using SSE\-KMS\)\. For both SSE\-S3 and KMS encryption, Athena is able to determine the proper materials to use to decrypt the dataset and create the table, so you don't need to provide key information\.

Users that run queries, including the user who creates the table, must have the appropriate permissions as described earlier\.

**Important**  
If you use Amazon EMR along with EMRFS to upload encrypted Parquet files, you must disable multipart uploads \(set `fs.s3n.multipart.uploads.enabled` to `false`\); otherwise, Athena is unable to determine the Parquet file length and a **HIVE\_CANNOT\_OPEN\_SPLIT** error occurs\. For more information, see [Configure Multipart Upload for Amazon S3](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-upload-s3.html#Config_Multipart) in the *EMR Management Guide*\.

Indicate that the dataset is encrypted in Amazon S3 in one of the following ways\. This is not required if SSE\-KMS is used\.

+ Use the [CREATE TABLE](create-table.md) statement with a `TBLPROPERTIES` clause that specifies `'has_encrypted_data'='true'`\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/athena/latest/ug/images/encrypt_has_encrypted.png)

+ Use the [JDBC driver](connect-with-jdbc.md) and set the `TBLPROPERTIES` value as above when you execute [CREATE TABLE](create-table.md) using `statement.executeQuery()`\.

+ Use the **Add table** wizard in the Athena console, and then choose **Encrypted data set** when you specify a value for **Location of input data set**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/athena/latest/ug/images/encrypt_has_encrypted_console.png)

Tables based on encrypted data in Amazon S3 appear in the **Database** list with an encryption icon\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/athena/latest/ug/images/encrypted_table_icon.png)

## Encrypting Query Results Stored in Amazon S3<a name="encrypting-query-results-stored-in-s3"></a>

You use the Athena console or JDBC driver properties to specify that query results, which Athena stores in the S3 staging directory, are encrypted in Amazon S3\. This setting applies to all Athena query results\. You can't configure the setting for individual databases, tables, or queries\.

**To encrypt query results stored in Amazon S3 using the console**

1. In the Athena console, choose **Settings**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/athena/latest/ug/images/settings.png)

1. For **Query result location**, enter a custom value or leave the default\. This is the Amazon S3 staging directory where query results are stored\.

1. Choose **Encrypt query results**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/athena/latest/ug/images/encrypt_query_results.png)

1. For **Encryption type**, choose **CSE\-KMS**, **SSE\-KMS**, or **SSE\-S3**\.

   If you chose **SSE\-KMS** or **CSE\-KMS**, for **Encryption key**, specify one of the following:

   + If your account has access to an existing KMS CMK, choose its alias, or

   + Choose **Enter a KMS key ARN** and then enter an ARN\.

   + To create a new KMS key, choose **Create KMS key**, use the IAM console to create the key, and then return to specify the key by alias or ARN as described in the previous steps\. For more information, see [Creating Keys](http://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.

1. Choose **Save**\.

## Encrypting Query Results stored in Amazon S3 Using the JDBC Driver<a name="jdbc-encryption"></a>

You can configure the JDBC Driver to encrypt your query results using any of the encryption protocols that Athena supports\. For more information, see [JDBC Driver Options](connect-with-jdbc.md#ate-jdbc-driver-options)\.