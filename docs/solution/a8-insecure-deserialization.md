# Insecure Deserialization

The `Legacy Bulk Import` feature at http://127.0.0.1:9090/app/bulkproducts?legacy=true does not securely deserialize the data thus allowing remote code execution.

![jse1](/resources/jse1.png)

The following input will trigger the vulnerability

```
{"rce":"_$$ND_FUNC$$_function (){require('child_process').exec('id;cat /etc/passwd', function(error, stdout, stderr) { console.log(stdout) });}()"}
```

![jse2](/resources/jse2.png)

**Vulnerable Code snippet**

*core/appHandler.js*
```         
...
module.exports.bulkProductsLegacy = function (req,res){
	// TODO: Deprecate this soon
	if(req.files.products){
		var products = serialize.unserialize(req.files.products.data.toString('utf8'))
...
```

**Solution**

Since the required feature is to essentially parse a JSON, it can be parsed securely using `JSON.parse` instead.

*core/appHandler.js*
```
...
module.exports.bulkProductsLegacy = function (req,res){
	// TODO: Deprecate this soon
	if(req.files.products){
		var products = JSON.parse(req.files.products.data.toString('utf8'))
...
```

**Fixes**

Implemented in the following files

- *core/appHandler.js*

**Recommendation**

- Use secure and recommended ways to implement application features
- Ensure that potentially vulnerable legacy features are't accessible

**Reference**

- <https://www.owasp.org/index.php/Top_10-2017_A8-Insecure_Deserialization>
- <https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/>