# puppet-heira
- Two different methods of storing variables that can be used in place of any attribute:
- The params pattern, and Hiera
- Refactor MySQL module that's been set up to use params pattern to Hiera.
- Understand both how params work, and how to set up Hiera to mimic that same behavior

# Create OS-specific Hiera data
1. From the puppet master, find the mysql module:
    $ cd /etc/puppetlabs/code/environments/production/modules/mysql
