# Deployment Documentation

At the moment these are more aggregated notes for production Apereo OAE deployments.

For more questions and information in other areas of setting up an Apereo OAE production clusters, please get in touch with the [OAE Developers list](http://collab.sakaiproject.org/mailman/listinfo/oae-dev) or send questions to [oae-dev@collab.sakaiproject.org](mailto:oae-dev@collab.sakaiproject.org).

## Etherpad Clustering

Etherpad doesn't support clustering natively, so we distribute individual collabdoc content items to dedicated pads randomly. The following components come into play:

`config.js`: The config.js file has a config.etherpad property which defines an array of hosts. The **ordering** of the hosts in this list defines its numeric index in the cluster. We have an example puppet template of a [production config.js for reference](https://github.com/oaeproject/puppet-hilary/blob/master/modules/hilary/templates/config.js.erb#L361).

`nginx.conf`: In your nginx (or http reverse-proxy) configuration, you'll need to provide a location block to proxy to each of these servers individually. An example nginx.conf template that has these blocks is available in the [project team puppet configurations](https://github.com/oaeproject/puppet-hilary/blob/master/modules/nginx/templates/user_tenant_nginx.conf.erb#L233).