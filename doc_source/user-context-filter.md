--------

--------

# Filtering on user context<a name="user-context-filter"></a>

Amazon Kendra enables you to filter a query response to remove documents from the result based on a user name and group membership\. You can filter search results by user token or user attribute\. 

## Filtering by user token<a name="context-filter-token"></a>

When a document is indexed into Amazon Kendra, a corresponding access control list \(ACL\) is ingested\. The ACL specifies which user names and group names are allowed or denied access to the document\. Documents without an ACL are public documents\. 

When you query the documents, you want to ensure the correct ACL is applied\. For example, you can use an Open ID token\. It uses a JSON format\. When you issue a query, Amazon Kendra extracts and validates the token\. Amazon Kendra pulls the user and group information and then runs the query\. All of the documents the user has access to including public documents are returned\. 

The following code shows how to include a user token\. 

```
response = kendra.query(
    QueryText = query,
    IndexId = index,
    UserToken = {
        Token = "token"
    })
```

## Filtering by user attribute<a name="context-filter-attribute"></a>

User attribute filtering requires you to: 
+ Set attributes on the indexed documents\.
+ Include user and group information when querying the index\.

**Warning**  
User context filtering requires you to provide user and group information\. If no user and group information is included in the query, Amazon Kendra returns all documents\. If you provide user and group information, only documents with matching attributes are returned\. Documents without attributes are not returned\. 

You can add a user or a group to either an allow list or a deny list for the document\. If a user or group is added to the deny list, the document is filtered out of any query that contains the user or group\.

You specify the user and group information when you query the index using the built\-in attributes `_user_id` and `_group_ids`\. You can set up to 100 group identifiers\.

Amazon Kendra does not match users to groups\. When you use user\-context filtering, you must include all of the groups that a user belongs to when you make the query\. For example, if a user belongs to two groups, "HR" and "IT," you must include both groups in the request\. 

The following example shows a request that filters the query response based on the user ID and groups\. The query will return any document that has the user or the "HR" or "IT" groups in the allow list\. If the user or either group is in the deny list for a document, the document is not returned\.

```
response = kendra.query(
        QueryText = query,
        IndexId = index,
        AttributeFilter = {
            "OrAllFilters": [
                {
                    "EqualsTo": {
                        "Key": "_user_id",
                        "Value": {
                            "StringValue": "user1"
                        }
                     }
                },
                {
                    "EqualsTo": {
                        "Key": "_group_ids",
                        "Value": {
                            "StringListValue": ["HR", "IT"]
                        }
                    }
                }
            ]
        }
        )
```

**Note**  
User context filtering isn't an authentication or authorization control for your content\. It doesn't do user authentication on the user and groups sent to the `Query` operation\. It is up to your application to ensure that the user and group information sent to `Query` operation is authenticated and authorized\.

There is an implementation of user context filtering for each data source\. The following section describes each implementation\. 

**Topics**
+ [Filtering by user token](#context-filter-token)
+ [Filtering by user attribute](#context-filter-attribute)
+ [User context filtering for documents added directly to an index](#context-filter-batch)
+ [User context filtering for frequently asked questions](#context-filter-faq)
+ [User context filtering for database data sources](#context-filter-jdbc)
+ [User context filtering for Confluence data sources](#context-filter-confluence)
+ [User context filtering for Google Drive data sources\.](#context-filter-google)
+ [User context filtering for Microsoft OneDrive data sources](#context-filter-onedrive)
+ [User context filtering for Amazon S3 data sources](#context-filter-s3)
+ [User context filtering for Salesforce data sources](#context-filter-salesforce)
+ [User context filtering for ServiceNow data sources](#context-filter-servicenow)
+ [User context filtering for Microsoft SharePoint data sources](#context-filter-sharepoint-online)

## User context filtering for documents added directly to an index<a name="context-filter-batch"></a>

To specify user context filters for a document that you add directly to an index, you provide the filter in the `AccessControlList` field of the document\. You provide three pieces of information:
+ The access that the entity should have\. You can say `ALLOW` or `DENY`\.
+ The type of entity\. You can say `USER` or `GROUP`\.
+ The name of the entity\.

You can add up to 200 entries in the `AccessControlList` field\.

## User context filtering for frequently asked questions<a name="context-filter-faq"></a>

You can add user context information to frequently asked questions when you use custom \.csv or JSON file formats\. For more information, see [Adding questions and answers directly to an index ](in-creating-faq.md)\.

## User context filtering for database data sources<a name="context-filter-jdbc"></a>

For a database data source, information for user context filtering comes from a column in the source table\. You specify the column name in the console or when you create the data source using the `AclConfiguration` field\. 

A database data source has the following limitations:
+ You can only specify an allow list for a database data source\. You can't specify a deny list\. 
+ You can only specify groups\. You can't specify individual users for the allow list\.
+ The database column should be string containing a semi\-colon delimited list of groups\.

## User context filtering for Confluence data sources<a name="context-filter-confluence"></a>

When you use a Confluence data source, Amazon Kendra gets user and group information from the Confluence instance\.

You configure user and group access to spaces using the space permissions page\. For pages and blogs, you use the restrictions page\. For more information about space permissions, see [ Space Permissions Overview ](https://confluence.atlassian.com/doc/space-permissions-overview-139521.html) on the Confluence Support website\. For more information about page and blog restrictions, see [ Page Restrictions ](https://confluence.atlassian.com/doc/page-restrictions-139414.html) on the Confluence Support website\.

The Confluence group and user names are mapped as follows:
+ `_group_ids` – Group names are present on spaces, pages, and blogs where there are restrictions\. They are mapped from the name of the group in Confluence\. Group names are always lower case\.
+ `_user_id` – User names are present on the space, page, or blog where there are restrictions\. They are mapped depending on the type of Confluence instance you are using\.
  + Server – The `_user_id` is the username\. The username is always lower case\.
  + Cloud – The `_user_id` is the account ID of the user\.

## User context filtering for Google Drive data sources\.<a name="context-filter-google"></a>

A Google Workspace Drive data source returns user and group information for Google Drive users and groups\. Group and domain membership are mapped to the `_group_ids` index field\. The Google Drive user name is mapped to the `_user_id` field\.

When you provide one or more user email addresses in the `Query` operation, only documents that have been shared with those email addresses are returned\. The following `AttributeFilter` parameter only returns documents shared with "martha@example\.com"\.

```
"AttributeFilter": {
                "EqualsTo":{
                   "Key": "_user_id", 
                   "Value": {
                       "StringValue": "martha@example.com"
                   }
               }
           }
```

If you provide one or more group email addresses in the query, only documents shared with the groups are returned\. The following `AttributeFilter` parameter only returns documents shared with the "hr@example\.com" group\.

```
"AttributeFilter": {
                "EqualsTo":{
                   "Key": "_group_ids", 
                   "Value": {
                       "StringListValue": ["hr@example.com"]
                   }
               }
           }
```

If you provide the domain in the query, all documents shared with the domain are returned\. The following `AttributeFilter` parameter returns documents shared with the "example\.com" domain\.

```
"AttributeFilter": {
                "EqualsTo":{
                   "Key": "_group_ids", 
                   "Value": {
                       "StringListValue": ["example.com"]
                   }
               }
           }
```

## User context filtering for Microsoft OneDrive data sources<a name="context-filter-onedrive"></a>

Amazon Kendra retrieves user and group information from Microsoft OneDrive when it indexes the documents on the site\. The user and group information is taken from the underlying Microsoft SharePoint site that hosts OneDrive\.

When you use a OneDrive user or group for user context filtering, calculate the ID as follows:

1. Get the site name\. For example, `https://host.onmicrosoft.com/sites/siteName.`

1. Take the MD5 hash of the site name\. For example, `430a6b90503eef95c89295c8999c7981`\.

1. Create the user email or group ID by concatenating the MD5 hash with a vertical bar \(\|\) and the ID\. For example, if a group name is "site owners", the group ID would be

   `"430a6b90503eef95c89295c8999c7981|site owners"`

   For the user name "someone@host\.onmicrosoft\.com," the user ID would be the following:

   `"430a6b90503eef95c89295c8999c7981|someone@host.onmicrosoft.com"`

Send the user or group ID to Amazon Kendra as the `_user_id` or `_group_ids` attribute when you call the [Query](API_Query.md) operation\. For example, the AWS CLI command that uses a group to filter the query response looks like this:

```
aws kendra  query \
                --index-id index ID  
                --query-text "query text" 
                --attribute-filter '{
                   "EqualsTo":{
                     "Key": "_group_ids", 
                     "Value": {"StringValue": "430a6b90503eef95c89295c8999c7981|site owners"}
                  }}'
```

## User context filtering for Amazon S3 data sources<a name="context-filter-s3"></a>

You add user context filtering to a document in an Amazon S3 data source using a metadata file associated with the document\. You add the information to the `AccessControlList` field in the JSON document\. For more information about adding metadata to the documents indexed from an Amazon S3 data source, see [S3 document metadata](s3-metadata.md)\.

You provide three pieces of information:
+ The access that the entity should have\. You can say `ALLOW` or `DENY`\.
+ The type of entity\. You can say `USER` or `GROUP`\.
+ The name of the entity\.

You can add up to 200 entries in the `AccessControlList` field\.

## User context filtering for Salesforce data sources<a name="context-filter-salesforce"></a>

A Salesforce data source returns user and group information from Salesforce access control list \(ACL\) entities\. The ProfileSet, PermissionSet, Group, and Role entities are mapped to the Amazon Kendra `_group_ids` index field\. The Salesforce User entity is mapped to the Amazon Kendra `_user_id` index field\.

The `_group_id` for a Salesforce chatter feed has two possible values\. If the feed is visible only to a Chatter or Collaboration group, that group is the Amazon Kendra `_group_id` value\. If not, Amazon Kendra creates a group named after the user that posted the chatter item\. All users that subscribed to the item are part of that group\.

## User context filtering for ServiceNow data sources<a name="context-filter-servicenow"></a>

User context filtering isn't currently supported for ServiceNow\.

## User context filtering for Microsoft SharePoint data sources<a name="context-filter-sharepoint-online"></a>

Amazon Kendra retrieves user and group information from Microsoft SharePoint when it indexes the documents on the site\. To filter your documents, provide user and group information when you call the `Query` operation\. 

To filter using a user name, use the user's email address\. For example, johnstiles@example\.com\.

When you use a SharePoint group for user context filtering, calculate the group ID as follows:

1. Get the site name\. For example, `https://host.onmicrosoft.com/sites/siteName.`

1. Take the MD5 hash of the site name\. For example, `430a6b90503eef95c89295c8999c7981`\.

1. Create the group ID by concatenating the MD5 hash with a vertical bar \(\|\) and the group name\. For example, if the group name is "site owners", the group ID would be

   `"430a6b90503eef95c89295c8999c7981|site owners"`

Send the group ID to Amazon Kendra as the `_group_ids` attribute when you call the [Query](API_Query.md) operation\. For example, the AWS CLI command looks like this:

```
aws kendra  query \
                --index-id index ID  
                --query-text "query text" 
                --attribute-filter '{
                   "EqualsTo":{
                     "Key": "_group_ids", 
                     "Value": {"StringValue": "430a6b90503eef95c89295c8999c7981|site owners"}
                  }}'
```