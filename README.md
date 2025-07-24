# Myrtle
Schema for documenting XML resources, such as XSLT stylesheets, XProc and Schematron.

## Usage Guide

### Documentation about a stylesheet or step 

The wrapper for documentation about a stylesheet or pipeline is `m:document`.  It's expected to include a brief description of the intended purpose of the stylesheet/pipeline and possibly some additional metadata, for example:

```
<m:document>
    <m:title>GEDCOM Text to XML</m:title>
    <m:desc>Converts the text serialisation of GEDCOM to very basic XML.</m:desc>
    <m:contributors>
    	<m:original-author>Honor Harrington</m:original-author>
    </m:contributors>
    <m:history created="2020-07-13" />
</m:document>
```  
Over time, this will likely grow richer, with the addition of general notes or a change history, for example:

```
<m:document xmlns:m="http://xylarium.org/ns/xml/documentation">
	<m:title>GEDCOM Text to XML</m:title>
	<m:desc>Converts the text serialisation of GEDCOM to very basic XML.</m:desc>
	<m:contributors>
		<m:original-author>Honor Harrington</m:original-author>
		<m:contributor>A. McKeon</m:contributor>
		<m:contributor>Estelle Matsuko</m:contributor>
	</m:contributors>
	<m:note date="2020-07-13">So far only tested on data exported from Ancestry.com.</m:note>
	<m:note>Current scope is only GEDCOM 5.5.1</m:note>	
	<m:note todo="true">Need to test with GEDCOM exported from other sources.</m:note>
	<m:history created="2020-07-13">
		<m:change date="2021-01-30" version="2.0.0">
			<m:desc>Refactored from XSLT 2.0 to 3.0</m:desc>
			<m:related href="https://www.w3.org/TR/xslt-30/" rel="specification">XSLT 3.0 Spec</m:related>
			<m:contributor>A. McKeon</m:contributor>
		</m:change>
		<m:change date="2020-08-02">
			<m:desc>
				<m:ul>
					<m:ingress>Assorted bugfixes:</m:ingress>
					<m:li><m:link rel="bug" href="https://jira.my.org/browse/ISSUE-123">Removed leading underscore from proprietary tags</m:link></m:li>
					<m:li>Added handling for lines that represent a cross-reference</m:li>
				</m:ul>
			</m:desc>
		</m:change>
	</m:history>
</m:document>
``` 

If you wish, you can also document top-level parameters, inputs and outputs:

```
<m:document>
	<m:desc>Creates fan chart visualisation of a person's direct ancestors.</m:desc>
	<m:input>A person's pedigree marked up using Sapling XML.</m:input>
	<m:param name="radius-of-core">The desired radius of the centre of the fan chart, in pixels.</m:param>
	<m:param name="max-generations">The maximum number of generations to include.</m:param>
	<m:return>A Scaleable Vector Graphic (SVG).</m:return> 
</m:document>
```

### XProc

There are a few extra considerations when writing documentation for an XProc step, for example, you will need to wrap the Myrtle documentation in a p:documentation element:

```
<p:documentation>
	<m:document>
		<m:desc>Creates a record of provenance metadata associated with a transformation.</m:desc>
		<m:input port="source">Source documents that were used during the transformation.</m:input>
		<m:param name="include-user-name">Used to control whether the result should include or exclude the name of the person who started the transformation (if available).</m:param>
		<m:param name="start-time">The date and time when the transformation process began.</m:param>
		<m:return>A prov:Bundle as an XML document</m:return> 
	</m:document>
</p:documentation>
```

A `p:option` is documented using `m:param`.

### Documentation about the component parts of a stylesheet or step

You can document an XSLT template or call to an XProc step by adding an `m:component` immediately before that item:
```
<m:component>Create an exact copy of the matched node or attribute.</m:component>
<xsl:template match="node() | attribute()">
	<xsl:copy>
		<xsl:apply-templates select="attribute(), node()" />
	</xsl:copy>
</xsl:template>
```
All it must include is a description (`m:desc`) so it can be very simple, per the example above, or, if you prefer, more detailed:
```
<m:component ref="next-generation" xmlns:m="http://xylarium.org/ns/xml/documentation">
	<m:title>Next Generation</m:title>
	<m:desc>
		<m:p>Create a segment for each of the parents of the people in the current generation.</m:p>
	</m:desc>
	<m:param name="current-generation">A sequence of person elements, one for each of the people in the generation of ancestors currently being processed.</m:param>
	<m:return>An svg:g (group) wrapping a ring of segments.</m:return>
</m:component>
<xsl:template name="next-generation">
	<xsl:param name="current-generation" as="element()*" tunnel="true" />
	
	<xsl:variable name="parents" select="$current-generation/parents/person[persona]" as="element()*" />
	<xsl:if test="count($parents) > 0">
		<xsl:call-template name="draw-generation-group">
			<xsl:with-param name="current-generation" select="$parents" as="element()*" tunnel="true" />
		</xsl:call-template>
	</xsl:if>
</xsl:template>
```
Like the detailed version of `m:document`, `m:component` can include:
* a title (`m:title`, optional)
* a description (`m:desc`, required)
* inputs (`m:input`, optional)
* parameters (`m:param`, optional)
* a description of the output(s) (`m:return`, optional)
* a list of contributors (`m:contributors`, optional)
* notes (`m:note`, optional)
* change history (`m:history`, optional)

You can also use `m:component` to document other children of the root element in a stylesheet, including (but not limited to):

* `xsl:include`
* `xsl:import`
* `xsl:param`
* `xsl:variable`
* `xsl:function`
* `xsl:key`

In an XProc step, `m:component` needs to wrapped inside a `p:documentation` element so can be used anywhere that a `p:documentation` element can go, for example, to document: 
* `p:import`
* `p:input`
* `p:output`
* `p:option`
* `p:variable`
* `p:group`
* `p:for-each`
* `p:viewport`
* `p:if`
* `p:choose`
* `p:try`
* `p:xslt`
* `p:invisible-xml`
* `p:identity`    

###  Comment

You can also use `m:comment` to document components.  It supports less metadata and is included simply to make it easier to add (and identify) a temporary note.  It can include:

  * either inline text mixed with links (`m:link`) and/or inline code (`m:code`)
  * or a mix of:
      * paragraphs (`m:p`)
      * lists (`m:ul` or `m:ol`)
      * code blocks (`m:code-block`) 

`m:comment` can be used anywhere that `m:component` can be used.

# Etymology
This project is named after the Myrtle tree.  It belongs to a collection of repositories that have tree-themed names (they're not all public).
