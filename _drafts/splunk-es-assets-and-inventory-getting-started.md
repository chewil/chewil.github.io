---
layout: post
title: Splunk ES Assets and Inventory, Getting Started
published: true
date: 2021-11-08 12:00:00
category: Splunk
tags: "Splunk ES" asset-inventory admin
---
*Opinions expressed are solely my own and do not express the views or opinions of my employer, of Splunk, nor anyone else...*  

The following is a collections of various notes I took while learning and setting up ES’s asset and identity database.  This information may not be applicable to all environments, so use them as references.  As of this writing (November, 2021) I am using Splunk Enterprise 8.2.2.1 with ES 6.4.  Future versions of ES may handle assets and inventory differently, however, and IMO, the methods and process should still be relevant.

# Why bother with ES Assets and Identity?
The immediate benefit after populating the ES asset and identity tables is the enrichment fields showing up for the `src`, `dest`, `dvc` and `user` fields.  Example of the additional enrichment fields are:  category, ip, nt_host, priority, user’s full name, user’s manager, etc. They're new fields created by a set of automatic lookups and they're named using the source field as a prefix such as:  src_ip, src_nt_host, src_mac, dest_mac, user_phone, user_mail, etc.

# Identify Data Sources
The definitive source for managing ES Assets and Identity is the "[Add asset and identity data to Splunk Enterprise Security](https://docs.splunk.com/Documentation/ES/latest/Admin/Addassetandidentitydata)" section of the official **Splunk Enterprise Security** document.

**Identity**: This is a table of users.  Documenation says to use the `|ldapquery` command to pull this data out of AD, but querying AD via LDAP can be extremely slow for a large enterprise.  

If you have the "**Splunk Supporting Add-on for Active Directory**" app from [splunk base](https://splunkbase.splunk.com/app/742/) deployed to your domain controllers, enable the ActiveDirectory sourcetype collection.  It will periodically poll the AD for recently changed objects and forward them to Splunk.  If the index is named "`msad`" then you can just query that index to update the identity table.

# Lookup Tables (KVStore)

1- `asset_lookup_by_str` - A list of assets.  This should be a list of all known networking capable hardware.  This include all company owned, vendor deployed and user BYOD devices that could be connected to your managed networks or VPN.
	- Note:  This list is kept up to date and completed by merging asset data from multiple sources.  The method used for the merging is custom and I will discuss in more details later.

2- `asset_lookup_by_cidr` - A subset of the `asset_lookup_by_str` lookup table containing only the rows where the `ip` field is in CIDR format that are 31 or less bits. I personally do not find this lookup table very useful, and have made some modifications that I will mention later.

3- `identity_lookup_expanded` - A list of identities, or user accounts from AD.

Try viewing these 3 lookup tables using the `| inputlookup` command to see what information are contained within each.  Splunk has a really good document on this topic on Splunk Docs on the topic of ["adding asset and identity data to Splunk ES"](https://docs.splunk.com/Documentation/ES/latest/Admin/Addassetandidentitydata).

In that doc, Splunk listed many data sources to gather asset and identity information from.  Those are great for starting up, however, it’s best to look at all source data collected in Splunk and look for any of them that contains combinations of hostname, FQDN, IP, MAC and user information.  

**Issue:**  TA’s from SplunkBase, even the ones mentioned in the Doc, rarely have all the fields extracted or named correctly.

- Expect to spend a LONG time tweaking the field extraction, alias, calculation and transformation rules.  

**Issue:**  There are saved searches in SA-IdentityManagement that will merge all the asset and identity sources into those 3 lookup tables above.  They do not, however, dedup the information, so only the first entry found will be returned by the lookup.  

- Modify ES's merge scripts to properly merge the relevant data and remove duplicates.

## Save Searches

Test
