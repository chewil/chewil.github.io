---
layout: post
title: Splunk ES Assets and Inventory, Getting Started
published: true
date: 2021-11-08 12:00:00
category: Splunk
tags: "Splunk ES" asset-inventory admin
---
*Opinions expressed are solely my own and do not express the views or opinions of my employer, of Splunk, nor anyone else...*  

The following are collections of various notes I took while learning and defining ES’s asset and identity information at work.  These methods may not be applicable to all environments, so use them as references.  As of this writing (November, 2021) I am using Splunk Enterprise 8.2.2.1 with ES 6.4.  Future versions of ES may handle assets and inventory different, however, and IMO, the methods and process should still be relevant.

# Why bother with ES Assets and Identity?
An immediate benefit after populating the ES asset and identity tables is the enrichment fields showing up for the `src`, `dest`, `dvc` and `user` fields.  Example of the additional enrichment fields are:  category, ip, nt_host, priority, user’s full name, user’s manager, etc.

The `category` field is an interesting one.  Think of it as ES's way of "tagging" assets and identities.  You may see references to "known_scanner" in the ES Admin guide, but, IMO, they don't do a good job explaining how that's used or how to even configure ES to identify an asset as a "known_scanner".  This "known_scanner" is really just tagging a known vulnerabilty scanner device by adding the value "known_scanner" to its `category` field.

The `ip` and `nt_host` field enrichments are very useful too!  A typical stateful firewall log shoudl show, at the minimal, the Layer3 session information containing the 5-tuples; src_ip, src_port, dest_ip, dest_port and protocol.  With asset enrichment, ES can fill in the latest known hostname (or FQDN) of the source and destination IP's found in, for example, DHCP logs.

For user accounts, the identity enrichment lookups can return the users's full name, email, manager, etc.  These are also valuable information for the SOC Analyst or the correaltion search to, for example, define a priority or severity.

So having a near complete and current asset and identity table is crucial from the SOC's perspective.  Hopefully there is enough useful information in here to help a Splunk ES administator setup and maintain their asset and identity table. 

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
