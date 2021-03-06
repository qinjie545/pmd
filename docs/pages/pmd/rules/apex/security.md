---
title: Security
summary: These rules deal with different security problems that can occur within Apex.
permalink: pmd_rules_apex_security.html
folder: pmd/rules/apex
sidebaractiveurl: /pmd_rules_apex.html
editmepath: ../pmd-apex/src/main/resources/rulesets/apex/security.xml
keywords: Security, ApexSharingViolations, ApexOpenRedirect, ApexInsecureEndpoint, ApexXSSFromURLParam, ApexXSSFromEscapeFalse, ApexBadCrypto, ApexCSRF, ApexSOQLInjection, ApexCRUDViolation, ApexDangerousMethods, ApexSuggestUsingNamedCred
---
## ApexBadCrypto

**Since:** PMD 5.5.3

**Priority:** Medium (3)

The rule makes sure you are using randomly generated IVs and keys for `Crypto` calls.
Hard-wiring these values greatly compromises the security of encrypted data.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexBadCryptoRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexBadCryptoRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    Blob hardCodedIV = Blob.valueOf('Hardcoded IV 123');
    Blob hardCodedKey = Blob.valueOf('0000000000000000');
    Blob data = Blob.valueOf('Data to be encrypted');
    Blob encrypted = Crypto.encrypt('AES128', hardCodedKey, hardCodedIV, data);
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexBadCrypto" />
```

## ApexCRUDViolation

**Since:** PMD 5.5.3

**Priority:** Medium (3)

The rule validates you are checking for access permissions before a SOQL/SOSL/DML operation.
Since Apex runs in system mode not having proper permissions checks results in escalation of 
privilege and may produce runtime errors. This check forces you to handle such scenarios.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexCRUDViolationRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexCRUDViolationRule.java)

**Example(s):**

``` java
public class Foo {
    public Contact foo(String status, String ID) {
        Contact c = [SELECT Status__c FROM Contact WHERE Id=:ID];

        // Make sure we can update the database before even trying
        if (!Schema.sObjectType.Contact.fields.Name.isUpdateable()) {
            return null;
        }

        c.Status__c = status;
        update c;
        return c;
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexCRUDViolation" />
```

## ApexCSRF

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Check to avoid making DML operations in Apex class constructor/init method. This prevents
modification of the database just by accessing a page.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexCSRFRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexCSRFRule.java)

**Example(s):**

``` java
public class Foo {
    public init() {
        insert data;
    }

    public Foo() {
        insert data;
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexCSRF" />
```

## ApexDangerousMethods

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Checks against calling dangerous methods.

For the time being, it reports:

* Against `FinancialForce`'s `Configuration.disableTriggerCRUDSecurity()`. Disabling CRUD security
opens the door to several attacks and requires manual validation, which is unreliable.
* Calling `System.debug` passing sensitive data as parameter, which could lead to exposure
of private data.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexDangerousMethodsRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexDangerousMethodsRule.java)

**Example(s):**

``` java
public class Foo {
    public Foo() {
        Configuration.disableTriggerCRUDSecurity();
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexDangerousMethods" />
```

## ApexInsecureEndpoint

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Checks against accessing endpoints under plain **http**. You should always use
**https** for security.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexInsecureEndpointRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexInsecureEndpointRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    void foo() {
        HttpRequest req = new HttpRequest();
        req.setEndpoint('http://localhost:com');
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexInsecureEndpoint" />
```

## ApexOpenRedirect

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Checks against redirects to user-controlled locations. This prevents attackers from
redirecting users to phishing sites.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexOpenRedirectRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexOpenRedirectRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    String unsafeLocation = ApexPage.getCurrentPage().getParameters.get('url_param');
    PageReference page() {
       return new PageReference(unsafeLocation);
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexOpenRedirect" />
```

## ApexSharingViolations

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Detect classes declared without explicit sharing mode if DML methods are used. This
forces the developer to take access restrictions into account before modifying objects.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexSharingViolationsRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexSharingViolationsRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    // DML operation here
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexSharingViolations" />
```

## ApexSOQLInjection

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Detects the usage of untrusted / unescaped variables in DML queries.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexSOQLInjectionRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexSOQLInjectionRule.java)

**Example(s):**

``` java
public class Foo {
    public void test1(String t1) {
        Database.query('SELECT Id FROM Account' + t1);
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexSOQLInjection" />
```

## ApexSuggestUsingNamedCred

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Detects hardcoded credentials used in requests to an endpoint.

You should refrain from hardcoding credentials:
  * They are hard to mantain by being mixed in application code
  * Particularly hard to update them when used from different classes
  * Granting a developer access to the codebase means granting knowledge
     of credentials, keeping a two-level access is not possible.
  * Using different credentials for different environments is troublesome
     and error-prone.

Instead, you should use *Named Credentials* and a callout endpoint.

For more information, you can check [this](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_named_credentials.htm)

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexSuggestUsingNamedCredRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexSuggestUsingNamedCredRule.java)

**Example(s):**

``` java
public class Foo {
    public void foo(String username, String password) {
        Blob headerValue = Blob.valueOf(username + ':' + password);
        String authorizationHeader = 'BASIC ' + EncodingUtil.base64Encode(headerValue);
        req.setHeader('Authorization', authorizationHeader);
    }
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexSuggestUsingNamedCred" />
```

## ApexXSSFromEscapeFalse

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Reports on calls to `addError` with disabled escaping. The message passed to `addError`
will be displayed directly to the user in the UI, making it prime ground for XSS
attacks if unescaped.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexXSSFromEscapeFalseRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexXSSFromEscapeFalseRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    Trigger.new[0].addError(vulnerableHTMLGoesHere, false);
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexXSSFromEscapeFalse" />
```

## ApexXSSFromURLParam

**Since:** PMD 5.5.3

**Priority:** Medium (3)

Makes sure that all values obtained from URL parameters are properly escaped / sanitized
to avoid XSS attacks.

**This rule is defined by the following Java class:** [net.sourceforge.pmd.lang.apex.rule.security.ApexXSSFromURLParamRule](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/java/net/sourceforge/pmd/lang/apex/rule/security/ApexXSSFromURLParamRule.java)

**Example(s):**

``` java
public without sharing class Foo {
    String unescapedstring = ApexPage.getCurrentPage().getParameters.get('url_param');
    String usedLater = unescapedstring;
}
```

**This rule has the following properties:**

|Name|Default Value|Description|
|----|-------------|-----------|
|cc_categories|[Style]|Code Climate Categories|
|cc_remediation_points_multiplier|1|Code Climate Remediation Points multiplier|
|cc_block_highlighting|false|Code Climate Block Highlighting|

**Use this rule by referencing it:**
``` xml
<rule ref="rulesets/apex/security.xml/ApexXSSFromURLParam" />
```

