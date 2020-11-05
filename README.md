# puppet-heira
- Two different methods of storing variables that can be used in place of any attribute:
- The params pattern, and Hiera
- Refactor MySQL module that's been set up to use params pattern to Hiera.
- Understand both how params work, and how to set up Hiera to mimic that same behavior

# Create OS-specific Hiera data
1. From the puppet master, find the mysql module:
- $ cd /etc/puppetlabs/code/environments/production/modules/mysql

2. Review all files, focusing specifically on the params.pp file. Make note of all parameters:
- install_name
- install_ensure
- config_path
- config_ensure
- service_name
- service_ensure
- service_enable
- service_hasrestart

- Notice how some variables are OS-family-specific while others are not.

3. Set up Hiera to filter by OS family:

- $ sudo vim hiera.yaml

  hierarchy:
 - name: 'Operating System Family'
   path: '%{facts.os.family}-family.yaml'

 - name: 'common'
   path: 'common.yaml'

- Save and Exit