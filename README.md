# Searchable DataObjects

Linkable DataObjects is a module that permit to link DataObjects from TinyMCE.

## Introduction

Pages are not always the better way to implement things. For example site news can grow rapidly and the first side effect
would be a big and difficult to manage SiteTree. DataObjects help maintaining things clean and straight, but unfortunately 
they are not included in frontend search. This module let you insert DataObject in search.

This module add the ability to link DataObjects through TinyMCE, just like internal pages. The linkable DataObjects must obviously implement 
a Link() function.

## Requirements

 * SilverStripe 3.1

### Installation

Install the module through [composer](http://getcomposer.org):

	composer require zirak/linkable-dataobject
  composer update

Make the DataObject implement Linkable interface (you need to implement Link(), LinkLabel(), link_shortcode_handler():

	:::php
	class DoNews extends DataObject implements Linkable {

		private static $db = array(
				'Title' => 'Varchar',
				'Subtitle' => 'Varchar',
				'News' => 'HTMLText',
				'Date' => 'Date',
		);
		private static $has_one = array(
				'Page' => 'PghNews'
		);

		/**
		* Link to this DO
		* @return string
		*/
	 public function Link() {
		 return $this->Page()->Link() . 'read/' . $this->ID;
	 }

	 /**
		* Label displayed in "Insert link" menu
		* @return string
		*/
	 public static function LinkLabel() {
		 return 'News';
	 }

	 /**
		* Replace a "[{$class}_link,id=n]" shortcode with a link to the page with the corresponding ID.
		* @param array  $arguments Arguments to the shortcode
		* @param string $content   Content of the returned link (optional)
		* @param object $parser    Specify a parser to parse the content (see {@link ShortCodeParser})
		* @return string anchor Link to the DO page
		*
		* @return string
		*/
	 static public function link_shortcode_handler($arguments, $content = null, $parser = null) {
		 if (!isset($arguments['id']) || !is_numeric($arguments['id'])) {
			 return;
		 }

		 $id =  $arguments['id'];
		 $page = DataObject::get_one(__CLASS__, "ID=$id");

		 if (!$page) {
			 $page = DataObject::get_one('ErrorPage', '"ErrorCode" = \'404\'');
			 return $page->Link();
		 }

		 if ($content) {
			 return sprintf('<a href="%s">%s</a>', $page->Link(), $parser->parse($content));
		 } else {
			 return $page->Link();
		 }
	 }
	}

Here you are a sample page holder, needed to implement the Link() function into the DataObject:

	:::php
	class PghNews extends Page {

		private static $has_many = array(
				'News' => 'DoNews'
		);

		public function getCMSFields() {
			$fields = parent::getCMSFields();

			/* News */
			$gridFieldConfig = GridFieldConfig_RelationEditor::create(100);
			// Remove unlink
			$gridFieldConfig->removeComponentsByType('GridFieldDeleteAction');
			// Add delete
			$gridFieldConfig->addComponents(new GridFieldDeleteAction());
			// Remove autocompleter
			$gridFieldConfig->removeComponentsByType('GridFieldAddExistingAutocompleter');
			$field = new GridField(
							'Faq', 'Faq', $this->News(), $gridFieldConfig
			);
			$fields->addFieldToTab('Root.News', $field);


			return $fields;
		}
	}

	class PghNews_Controller extends Page_Controller {

		private static $allowed_actions = array(
				'read'
		);

		public function read(SS_HTTPRequest $request) {
			$arguments = $request->allParams();
			$id = $arguments['ID'];

			// Identifico la faq dall'ID
			$Object = DataObject::get_by_id('DoNews', $id);

			if ($Object) {
				//Popolo l'array con il DataObject da visualizzare
				$Data = array($Object->class => $Object);
				$this->data()->Title = $Object->Title;

				$themedir = $_SERVER['DOCUMENT_ROOT'] . '/' . SSViewer::get_theme_folder() . '/templates/';
				$retVal = $this->Customise($Data);
				return $retVal;
			} else {
				//Not found
				return $this->httpError(404, 'Not found');
			}
		}
	}

Flush your cache and start linking your DataObjects.

### Suggested modules

 * Searchable DataObjects: http://addons.silverstripe.org/add-ons/zirak/searchable-dataobjects