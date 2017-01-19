#Install JWT Apex classes
```
Install JWT.apex and JWTBearer.apex from https://github.com/salesforceidentity/jwt
Install Vision.apex and HttpFormBuilder.apex
```

#Visualforce Page example
```java
public class VisionController {

    public String getAccessToken() {
        JWT jwt = new JWT('RS256');
        // jwt.cert = 'JWTCert'; // If you used a Salesforce certificate to sign up for a Predictive Services account
        jwt.pkcs8 = 'MIICXQIBAAKBgQC4U4Bma7kKa0CLU... contents from "predictive_services.pem" RSA private Key';
        jwt.iss = 'developer.force.com';
        jwt.sub = 'yourname@example.com';
        jwt.aud = 'https://api.metamind.io/v1/oauth2/token';
        jwt.exp = '3600';
        String access_token = JWTBearerFlow.getAccessToken('https://api.metamind.io/v1/oauth2/token', jwt);
        return access_token;    
    }
     
    // Alternate way to generate Access Token
    // You can upload the `predictive_services.pem` into your Salesforce org as `File` sobject and read it as below.
    
    public String getAccessTokenFromFile() {        
        ContentVersion base64Content = [SELECT Title, VersionData FROM ContentVersion where Title='predictive_services' LIMIT 1];
        String keyContents = base64Content.VersionData.tostring();
        keyContents = keyContents.replace('-----BEGIN RSA PRIVATE KEY-----', '');
        keyContents = keyContents.replace('-----END RSA PRIVATE KEY-----', '');
        keyContents = keyContents.replace('\n', '');
        jwt.pkcs8 = keyContents;

        // Get a new token
        JWT jwt = new JWT('RS256');
        jwt.pkcs8 = keyContents;
        jwt.iss = 'developer.force.com';
        jwt.sub = 'yourname@example.com';
        jwt.aud = 'https://api.metamind.io/v1/oauth2/token';
        jwt.exp = '3600';
        String access_token = JWTBearerFlow.getAccessToken('https://api.metamind.io/v1/oauth2/token', jwt);
        return access_token;    
    }

    public List<Vision.Prediction> getCallVisionUrl() {
        // Get a new token
        String access_token = getAccessToken();
        // or String access_token = getAccessTokenFromFile();
    
        // Make a prediction using URL to a file
        return Vision.predictUrl('http://metamind.io/images/generalimage.jpg',access_token,'GeneralImageClassifier');
    }

    public List<Vision.Prediction> getCallVisionContent() {
        // Get a new token
        String access_token = getAccessToken();
        // or String access_token = getAccessTokenFromFile();

        // Make a prediction for an image stored in Salesforce
        // by passing the file as blob which is then converted to base64 string
        ContentVersion content = [SELECT Title,VersionData FROM ContentVersion where Id = '06841000000LkfCAAS' LIMIT 1];
        return Vision.predictBlob(content.VersionData, access_token, 'GeneralImageClassifier');
    }
}
```

```xml
<apex:page Controller="VisionController">
  <apex:form >
  <apex:pageBlock >
      <apex:image url="http://metamind.io/images/generalimage.jpg">
      </apex:image>
      <br/>
      <apex:repeat value="{!AccessToken}" var="accessToken">
          Access Token:<apex:outputText value="{!accessToken}" /><br/>
    </apex:repeat>
      <br/>
      <apex:repeat value="{!callVisionUrl}" var="prediction">
          <apex:outputText value="{!prediction.label}" />:<apex:outputText value="{!prediction.probability}" /><br/>
      </apex:repeat>
  </apex:pageBlock>
<!--  <apex:pageBlock > -->
<!--      <apex:repeat value="{!callVisionContent}" var="prediction"> -->
<!--          <apex:outputText value="{!prediction.label}" />:<apex:outputText value="{!prediction.probability}" /><br/> -->
<!--    </apex:repeat> -->
<!--  </apex:pageBlock> -->
  </apex:form>
</apex:page>
```
