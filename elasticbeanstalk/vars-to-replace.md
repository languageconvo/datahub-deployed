Elasticsearch host url:

{{{ $elastic_host }}} = 1234abcd.us-west-2.aws.found.io

Elasticsearch username:

{{{ $elastic_username }}} = elastic

Elasticsearch password:

{{{ $elastic_password }}} = password123

MySQL host url:

{{{ $mysql_host }}} = datahub-mysql123.us-west-2.rds.amazonaws.com

MySQL username:

{{{ $mysql_username }}} = username

MySQL password:

{{{ $mysql_password }}} = password123

**Important**: this password gets passed into a Linux-based container as a variable, so do not use `$` in it (otherwise Linux will misread it). Using only alphanumeric characters might be a good idea, but this *is* your datahub root user's password, so make it really strong.

{{{ $datahub_master_password }}} = password456
