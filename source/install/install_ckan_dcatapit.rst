.. _install_ckan_dcatapit:


###################################
Installing the DCAT-AP_IT extension
###################################

============
Introduction
============

CKAN extension for the Italian Open Data Portals (DCAT_AP-IT).

====================
DCAT-AP_IT extension
====================

Overview 
--------

This extension provides plugins that allow CKAN to expose and consume metadata from other catalogs using RDF documents serialized according to the Italian DCAT Application Profile. The Data Catalog Vocabulary (DCAT) is "an RDF vocabulary designed to facilitate interoperability between data catalogs published on the Web".

.. warning:: This extension has been developed in 2016 to allow Ckan to manage dataset in accordance to the DCAT-AP-It specifications. A migration documentation provides various steps to integrate the ckanext-dcatapit extension into the existing environment.
.. warning:: If you're running old installation, you should read :ref:`Extension upgrade guide <dcatapit-upgrade>` section.

Requirements
------------

The ckanext-dcatapit extension has been developed for CKAN 2.4 or later and is based on the ckanext-dcat plugin. In order to use the dcatapit CSW harvester functionalities, you need to install also the ckanext-spatial extension.


PREPARING THE SYSTEM FOR DCAT_AP-IT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This ckanext-provbz has been updated to integrate the old version of this extension with ckanext-dcatapit:

	Follow the steps below only the first time you prepare the system for installing the ckanext-dcatapit.

1. Backup the DB ckan:
	
		su postgres

		pg_dump -U postgres -i ckan > ckan.dump
		
2. Upgrade the python setuptools version installed:

		pip install --upgrade setuptools

3. Install the ckanext-dcat extension:

		. /usr/lib/ckan/default/bin/activate

		cd /usr/lib/ckan/default/src

		git clone https://github.com/ckan/ckanext-dcat.git

		cd ckanext-dcat

		pip install -e .

		pip install -r requirements.txt
		
	- Edit the `/etc/ckan/default/production.ini` adding plugins:
	
		ckan.plugins = dcat dcat_rdf_harvester dcat_json_harvester dcat_json_interface

4. Update the ckanext-multilang extension:

		. /usr/lib/ckan/default/bin/activate

		cd /usr/lib/ckan/default/src/ckanext-multilang

		git pull 

		pip install -e .
		
5. Update the ckanext-provbz extension:

		. /usr/lib/ckan/default/bin/activate

		cd /usr/lib/ckan/default/src/ckanext-provbz

		git pull 

		git checkout dcatapit

		pip install -e .
	
	- Update the `/etc/ckan/default/production.ini` file adding the property below:
	
			ckan.i18n_directory = /usr/lib/ckan/default/src/ckanext-provbz/ckanext/provbz/translations
			
	- Change the name of the `provbz_theme` plugin to `provbz` (you can find that in the `ckan.plugins` property)
	
6. Install the ckanext-dcatapit following the steps reported in the next paragraph:
		
7. Update the ckanext-spatial extension:

		. /usr/lib/ckan/default/bin/activate

		cd /usr/lib/ckan/default/src/ckanext-spatial

		git checkout master

		git pull 

		pip install -e .
		
8. Run the SQL migration script to update DB tables (make sure to have rights to execute the sql file as user postgres):

		su postgres

		psql -U postgres -d ckan -f /usr/lib/ckan/default/src/ckanext-provbz/migration/sql/migration.sql
	
9. Restart CKAN

10. Rebuild the Solr indexes:

		. /usr/lib/ckan/default/bin/activate

		paster --plugin=ckan search-index rebuild  -c /etc/ckan/default/production.ini
		

 .. _dcatapit-installation:

Installation 
------------       

1. Install the **ckanext-dcatapit** extension.

As user ``ckan``::

	   $ . /usr/lib/ckan/default/bin/activate
	   (default)$ cd /usr/lib/ckan/default/src
	   (default)$ git clone https://github.com/geosolutions-it/ckanext-dcatapit.git
	   (default)$ cd ckanext-dcatapit
	   (default)$ pip instal -e .
	   
Enable the required plugins in your ini file::

		ckan.plugins = [...] dcatapit_pkg dcatapit_org dcatapit_config

In order to enable also the RDF harvester add ``dcatapit_harvester`` to the ``ckan.plugins`` setting in your CKAN::

		ckan.plugins = [...] dcatapit_pkg dcatapit_org dcatapit_config dcatapit_harvester

In order to enable also the CSW harvester add ``dcatapit_csw_harvester`` to the ``ckan.plugins`` setting in your CKAN::

		ckan.plugins = [...] dcatapit_pkg dcatapit_org dcatapit_config dcatapit_harvester dcatapit_csw_harvester

2. Enable the dcatapit profile adding the following configuration property in the ``production.ini`` file::

		`ckanext.dcat.rdf.profiles = euro_dcat_ap it_dcat_ap`

3. Configure the CKAN base URI::

		`ckanext.dcat.base_uri = YOUR_BASE_URI`

4. Initialize the CKAN DB with the mandatory table needed for localized vocabulary voices::

		`paster --plugin=ckanext-dcatapit vocabulary initdb --config=/etc/ckan/default/production.ini`

5. Then restart CKAN to make it load this new extensions.
     
6. The EU controlled vocabularies must be populated before start using the dcatapit plugin. Execute in sequence these commands::

		paster --plugin=ckanext-dcatapit vocabulary load --url http://publications.europa.eu/mdr/resource/authority/language/skos/languages-skos.rdf --name languages --config=/etc/ckan/default/production.ini
    
		paster --plugin=ckanext-dcatapit vocabulary load --url http://publications.europa.eu/mdr/resource/authority/data-theme/skos/data-theme-skos.rdf --name eu_themes --config=/etc/ckan/default/production.ini
    
		paster --plugin=ckanext-dcatapit vocabulary load --url http://publications.europa.eu/mdr/resource/authority/place/skos/places-skos.rdf --name places --config=/etc/ckan/default/production.ini
    
		paster --plugin=ckanext-dcatapit vocabulary load --url http://publications.europa.eu/mdr/resource/authority/frequency/skos/frequencies-skos.rdf --name frequencies --config=/etc/ckan/default/production.ini
    
		paster --plugin=ckanext-dcatapit vocabulary load --url http://publications.europa.eu/mdr/resource/authority/file-type/skos/filetypes-skos.rdf  --name filetype --config=/etc/ckan/default/production.ini
	
	
DCAT_AP-IT CSW Harvester
------------------------

The ckanext-dcatapit extension provides also a CSW harvester built on the **ckanext-spatial** extension, and inherits all of its functionalities. With this harvester you can harvest dcatapit dataset fields from the ISO metadata. The CSW harvester uses a default configuration usefull for populating mandatory fields into the source metadata, this json configuration can be customized into the harvest source form (please see the default one `into the harvester file <https://github.com/geosolutions-it/ckanext-dcatapit/blob/master/ckanext/dcatapit/harvesters/csw_harvester.py#L54>`_ ).

Below an example of the available configuration properties (for any configuration property not specified, the default one will be used)::

    {
       "dcatapit_config":{
          "dataset_themes":"OP_DATPRO",
          "dataset_places":"ITA_BZO",
          "dataset_languages":"{ITA,DEU}",
          "frequency":"UNKNOWN",
          "agents":{
             "publisher":{
                "code":"p_bz",
                "role":"publisher",
                "code_regex":{
                   "regex":"\\(([^)]+)\\:([^)]+)\\)",
                   "groups":[2]
                },
                "name_regex":{
                   "regex":"([^(]*)(\\(IPa[^)]*\\))(.+)",
                   "groups":[1, 3]
                }
             },
             "owner":{
                "code":"p_bz",
                "role":"owner",
                "code_regex":{
                   "regex":"\\(([^)]+)\\:([^)]+)\\)",
                   "groups":[2]
                },
                "name_regex":{
                   "regex":"([^(]*)(\\(IPa[^)]*\\))(.+)",
                   "groups":[1, 3]
                }
             },
             "author":{
                "code":"p_bz",
                "role":"author",
                "code_regex":{
                   "regex":"\\(([^)]+)\\:([^)]+)\\)",
                   "groups":[2]
                },
                "name_regex":{
                   "regex":"([^(]*)(\\(IPa[^)]*\\))(.+)",
                   "groups":[1, 3]
                }
             }
          },
          "controlled_vocabularies":{
             "dcatapit_skos_theme_id":"theme.data-theme-skos",
             "dcatapit_skos_places_id":"theme.places-skos"
          }
       }
    }

* ``dataset_themes``: default value to use for the dataset themes field if the thesaurus keywords are missing in the ISO metadata. The source metadata should have thesaurus keywords from the EU controlled vocabulary (data-theme-skos.rdf). Multiple values must be set between braces and comma separated values.

* ``dataset_places``: default value to use for the dataset geographical name field if the thesaurus keywords are missing in the ISO metadata. The source metadata should have thesaurus keywords from the EU controlled vocabulary (places-skos.rdf). Multiple values must be set between braces and comma separated values.

* ``dataset_languages``: default value to use for the dataset languages field. Metadata languages are harvested by the che ckanext-spatial extension (see the 'dataset-language' in iso_values). Internally the harvester map the ISO languages to the mdr vocabulary languages. The default configuration for that can be overridden in harvest source configuration by using an additional configuration property, like::

        "mapping_languages_to_mdr_vocabulary": {
            "ita': "ITA",
            "ger': "DEU",
            "eng': "ENG"
        }
        
* ``frequency``: default value to use for the dataset frequency field. Metadata frequencies are harvested by the che ckanext-spatial extension (see the 'frequency-of-update' in iso_values). Internally the harvester automatically map the ISO frequencies to the mdr vocabulary frequencies.

* ``agents``: Configuration for harvesting the dcatapit dataset agents from the responsible party metadata element. Below more details on the agent configuration::

         "publisher":{
            "code":"p_bz",      --> the IPA/IVA code to use as default for the agent identifier
            "role":"publisher", --> the responsible party role to harvest for this agent
            "code_regex":{      --> a regular expression to extrapolate a substring from the responsible party organization name
               "regex":"\\(([^)]+)\\:([^)]+)\\)",
               "groups":[2]     --> optional, dependes by the regular expression
            },
            "name_regex":{      --> a regular expression to extrapolate the IPA/IVA code from the responsible party organization name
               "regex":"([^(]*)(\\(IPA[^)]*\\))(.+)",
               "groups":[1, 3]  --> optional, dependes by the regular expression
            }
         }
     
* ``controlled_vocabularies``: To harvest 'dataset_themes' and 'dataset_places' the harvester needs to know the thesaurus ID or TITLE as specified into the source metadata.

.. note:: The default IPA code to use is extrapolated by the metadata identifier in respect to the RNDT specifications (ipa_code:UUID). This represents a last fallback if the agent regex does not match any code and if the agent code has not been specified in configuration.

Harvest source configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to set the dcatapit CSW harvester:

1. Specify a valid csw endpoint in the URL field 
2. Specify a title and a description for the harvest source
3. Select 'DCAT_AP-IT CSW Harvester' as source type
4. Provide your own configuration to override the default one

CSW Metadata Guidelines
^^^^^^^^^^^^^^^^^^^^^^^

* The dataset unique identifier will be harvested from the metadata fileIdentifier (see the above paragraph for additional notes about the IPA code).

* In order to harvest dcatapit dataset themes, the source metadata should have thesaurus keywords from the EU controlled vocabulary (data-theme-skos.rdf). Then the thesaurus identifier or title must be specified into the controlled_vocabularies->dcatapit_skos_theme_id configuration property

* In order to harvest dcatapit dataset geographical names, the source metadata should have thesaurus keywords from the EU controlled vocabulary (places-skos.rdf). Then the thesaurus identifier or title must be specified into the controlled_vocabularies->dcatapit_skos_places_id configuration property

* The dcatapit agents (publisher, holder, creator) will be harvested from the responsible party with the role specified in configuration (see 'agents' configuration property explained above)

* The dataset languages are harvested using the xpaths reported `into the ckanext-spatial harvested metadata file <https://github.com/ckan/ckanext-spatial/blob/master/ckanext/spatial/model/harvested_metadata.py#L723>`_

* The dataset frequency of update is harvested using the xpath reported `into the harvested metadata file <https://github.com/ckan/ckanext-spatial/blob/master/ckanext/spatial/model/harvested_metadata.py#L597>`_

Extending the package schema in your own extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: This paragraph describes, if you want, how the package schema can be extended by your own ckan extension, leveraging on the ckanext-dcatapit functionalities.

The dcatapit extension allows to define additional custom fields to the package schema by implementing the `ICustomSchema` interface 
in you CKAN extension. Below a sample::

    class ExamplePlugin(plugins.SingletonPlugin):

        # ICustomSchema
        plugins.implements(interfaces.ICustomSchema)

        def get_custom_schema(self):
            return [
                {
                    'name': 'custom_text',
                    'validator': ['ignore_missing'],
                    'element': 'input',
                    'type': 'text',
                    'label': _('Custom Text'),
                    'placeholder': _('custom texte here'),
                    'is_required': False,
                    'localized': False
                }
            ]

Through this an additional schema field named `custom_text` will be added to the package schema and automatically managed by the dcatapit extension. Below a brief description of the fields properties that can be used:

* ``name``: the name of the field
* ``validator``: array of validators to use for the field
* ``element``: the element type to use into the package edit form (ie. see the available ckan macros or macros defined into the dcatapit extension `here <https://github.com/geosolutions-it/ckanext-dcatapit/blob/master/ckanext/dcatapit/templates/macros/dcatapit_form_macros.html>`_
* ``type``: the type of input eg. email, url, date (default: text)
* ``label``: the human readable label
* ``placeholder``: some placeholder text
* ``is_required``: boolean of whether this input is requred for the form to validate
* ``localized``: True to enable the field localization by the dcatapit extension (default False). This need the ckanext-multilang installed.

Managing translations
^^^^^^^^^^^^^^^^^^^^^

The dcatapit extension implements the ITranslation CKAN's interface so the translations procedure of the GUI elements is automatically covered using the translations files provided in the i18n directory.

.. note:: Pay attention that the usage of the ITranslation interface can work only in CKAN 2.5 or later, if you are using a minor version of CKAN the ITranslation's implementation will be ignored.

Creating a new translation
--------------------------

.. note:: The steps below can be used only if you have to update existing translations files.

To create a new translation proceed as follow:

1. Extract new messages from your extension updating the pot file::

		python setup.py extract_messages
     
2.  Create a translation file for your language (a po file) using the existing pot file in this plugin::

		python setup.py init_catalog --locale YOUR_LANGUAGE

Replace YOUR_LANGUAGE with the two-letter ISO language code (e.g. es, de).
     
3. Do the translation into the po file

4. Once the translation files (po) have been updated, either manually or via Transifex, compile them by running::

		python setup.py compile_catalog --locale YOUR_LANGUAGE
     
Updating an existing translation
--------------------------------

In order to update the existing translations proceed as follow:

1. Extract new messages from your extension updating the pot file::
	
		python setup.py extract_messages
     
2. Update the strings in your po file, while preserving your po edits, by doing::

		python setup.py update_catalog --locale YOUR-LANGUAGE

3. Once the translation files (po) have been updated adding the new translations needed, compile them by running::

		python setup.py compile_catalog --locale YOUR_LANGUAGE

.. _dcatapit-upgrade:

=================
Extension upgrade
=================

DCAT_AP-IT Extension underwent significant modifications in various areas in the year 2018, especially in internal data format for various fields stored in database. Older installations may not display correcly some of extension-specific fields after straight code upgrade. In order to preserve existing data from older installation, you should run upgrade script that will convert old values to new format.

1. Perform database dump (this is a safety measure, "just in case")::

        su postgres
    	pg_dump -U postgres -i ckan > ckan.dump
	pg_dump -U postgres -i datastore > datastore.dump

2. Update extension code (both **ckanext-dcatapit** and **ckanext-multilang** need to be updated)::

        git pull

3. Update the Solr schema, ensure that following fields are present in `schema.xml`, **then restart Solr**::

        <field name="dcat_theme" type="string" indexed="true" stored="false" multiValued="true"/>
        <field name="dcat_subtheme" type="string" indexed="true" stored="false" multiValued="true"/>
        <dynamicField name="dcat_subtheme_*" type="string" indexed="true" stored="false" multiValued="true"/>
        <dynamicField name="organization_region_*" type="string" indexed="true" stored="false" multiValued="true"/>
        <dynamicField name="resource_license_*" type="string" indexed="true" stored="false" multiValued="true"/>
        <field name="resource_license" type="string" indexed="true" stored="false" multiValued="true"/>


4. Ensure that all the configuration properties required by the new version have been properly provided in .ini file (see `Installation <https://github.com/geosolutions-it/ckanext-dcatapit#installation>`_ paragraph).

Below the main involved configuration is reported::

	## Plugins Settings

	# Note: Add ``datastore`` to enable the CKAN DataStore
	#       Add ``datapusher`` to enable DataPusher
	#		Add ``resource_proxy`` to enable resorce proxying and get around the
	#		same origin policy
	ckan.plugins = resource_proxy datastore harvest ckan_harvester spatial_metadata spatial_query csw_harvester geonetwork_harvester stats text_view image_view recline_view pdf_view multilang multilang_harvester shibboleth pages dcat dcat_rdf_harvester dcat_json_harvester dcat_json_interface external_resource_list status_reports report provbz provbz_harvester datapusher **dcatapit_pkg dcatapit_org dcatapit_config dcatapit_theme_group_mapper dcatapit_ckan_harvester dcatapit_harvest_list dcatapit_harvester dcatapit_csw_harvester**

	## Old DCATAPIT Settings ------------
	ckanext.dcat.rdf.profiles = euro_dcat_ap it_dcat_ap
	ckanext.dcat.base_uri = http://test-dati.retecivica.bz.it

	## New DCATAPIT Settings ------------

	ckanext.dcat.expose_subcatalogs = False
	ckanext.dcat.clean_tags = True
	ckanext.dcatapit.form_tabs = True
	ckanext.dcatapit.localized_resources = True

	#ckanext.dcatapit.theme_group_mapping.file = /etc/ckan/theme_to_group.ini
	#ckanext.dcatapit.nonconformant_themes_mapping.file = /etc/ckan/topics.json

	geonames.username = XXX
	geonames.limits.countries = IT

5. Activate the virtual environment::
	
	. /usr/lib/ckan/default/bin/activate

6. Run model update::

        paster --plugin=ckanext-dcatapit vocabulary initdb --config=/etc/ckan/default/production.ini

7. Run vocabulary load commands (regions, licenses and sub-themes)::

        wget "https://raw.githubusercontent.com/italia/daf-ontologie-vocabolari-controllati/master/VocabolariControllati/territorial-classifications/regions/regions.rdf" -O "/tmp/regions.rdf"
        paster --plugin=ckanext-dcatapit vocabulary load --filename "/tmp/regions.rdf" --name regions --config "/etc/ckan/default/production.ini"
        wget "https://raw.githubusercontent.com/italia/daf-ontologie-vocabolari-controllati/master/VocabolariControllati/licences/licences.rdf" -O "/tmp/licenses.rdf"
        paster --plugin=ckanext-dcatapit vocabulary load --filename "/tmp/licenses.rdf" --name licenses --config "/etc/ckan/default/production.ini"
        paster --plugin=ckanext-dcatapit vocabulary load --filename "ckanext-dcatapit/examples/eurovoc_mapping.rdf" --name subthemes --config "/etc/ckan/default/production.ini" "ckanext-dcatapit/examples/eurovoc.rdf"

8. Run data migration command::

        paster --plugin=ckanext-dcatapit vocabulary migrate_data --config=/etc/ckan/default/production.ini > migration.log

 You can review migration results by viewing `migration.log` file. It will contain list of messages generated during migration. 

 Migration script will:

 * update all organizations and assign temporary identifier in form of `tmp_ipa_code_X` (where `X` is a number in sequence). Organization identifier is required field now, and thus temporary value is created to avoid errors in validation. Script will report each organization which have updated identifier in log with message similar to following: `org: [pab-foreste] PAB: Foreste : setting temporal identifier: tmp_ipa_code_X`

 * update all packages and migrate DCAT_AP-IT fields. Where possible, it will try to transform those fields into new notation/format. Successful package data migration will be marked with message like this::

        ---------
        updating ortofoto-di-merano-2005
        ---------

 If migration of specific field is not possible for some reason, or conversion will not be clean, there will be a message like this::
  
    	dataset test-dataset: the same temporal coverage start/end: 01-01-2014/01-01-2014, using start only
    	dataset test-dataset: no identifier. generating new one
    	dataset test-dataset: invalid modified date Manuelle. Using now timestamp
    	updating b36e6f42-d0eb-4b53-8e41-170c50a2384c occupati-e-disoccupati
    	---------
	
9. Rebuild Solr indexes::

		paster --plugin=ckan search-index rebuild -c /etc/ckan/default/production.ini
		
10. Restart Ckan

Field conversion notes
----------------------

* `conforms_to` is more complex structure now. It contains identifier, title and description. Converter will use old string value as an identifier of standard, and if multilang values are present, they will populate description subfield of standard. In case of multilang values present, Italian translation will be used as identifier.

* `creator` is a list of entities. It's composed of `creator_name` and `creator_identifier`, and converter will use existing values (including multilang name)

* `temporal_coverage` is a list of entries, where each entry is constructed from two old fields: `temporal_start` and `temporal_end`. If both values are equal, only `temporal_start` will be used. Some values may not be parseable, and should be adjusted manually in dataset.

* `theme` is required now, so if dataset lacks theme(s), default one (`OP_DATPRO`) will be assigned. Subthemes will be empty.

* `identifier` is required now. If it's missing, new one (UUID) will be generated.

* `modified` is required now. If it's missing or invalid, current date will be used.

* `frequency` is required now. If it's missing or invalid `UNKNOWN` value will be used.

* `holder_name` and `holder_identifier` behaves differently in new DCAT_AP-IT version. When dataset is created locally (wasn't harvested), rights holder information is gathered directly from organization to which dataset belongs. Organization is the source of `holder_name` and `holder_identifier` fields (including multilang name). However, harvested datasets will preserve original holder information that is attached to dataset. 


==================
Document changelog
==================

 .. csv-table:: Changelog
    :header: "Version", "Date", "Author", "Notes"

    1.0,,,Initial revision
    1.1,2018-08-29,CS,Migration section
