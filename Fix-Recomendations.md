
# Summary

The vulnerability is caused by the dependent on session-based rate limitation, absence of server-side input validation, and lack of CAPTCHA verification on the backend.

To remediate the issue, server-side controls must be implemented to prevent automated and malformed submission, regardless of client side behavior.

# Recommendations

## Enforce server-side CAPTCHA Validation 

Ensure Cloudflare Tunrstile tokens are verified on every submission

Reject requests when
- CAPTCHA field is mission
- CAPTCHA token is empty
- CAPTCHA verification fails

Do not accept any form submission without successful CAPTCHA validation
#### Realization example
```Js
import fetch from "node-fetch";
asycn function verifyCaptcha(token){
	const response = await fetch ("https://challenges.clouflare.com/turnstile/v1/siteverify",{
	method: "POST",
	headers: {"Content-type":"application/x-www-form-urlencoded"},
	body: new URLSearchParams({
	secret: process.env.TURNSTIE_SECRET_KEY, 
	response: token
	})
	});
	const data = await response.json();
	return data.success;
}

app.post("/form-submit", async (req, res, next) => {
const captchaToken = req.body["cf-turnstile-response"];
if (!captchaToken || !(await verifyCaptcha(captchaToken)))
	return res.status(400).json({error:"CAPTCHA validation failed" });
next();
} );
```
## Implement Persistent Rate Limitation

Replace session based validation with IP based on fingerprint-based limiting 
Limits should apply across sessions and page refreshes

Example

- Max 5 submissions per minute per IP
- Max 50 submissions per day per IP

Requests going over the limits should return HTTP 429 
#### Realization example
```Js
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";
import Redis from "ioredis";

const redisCLient = new Redis();
// 5 requests per minute
const limiter = rateLimit({
store: new RedisStore ({sendCommand:(args) => redisClient.call(args) })
windowMs:60*1000,
max:5,
standardHeaders:true,
legacyHeaders:flase,
message: " Too many requests, please try again later"
});

app.use("/form-submit", limiter);

```
## Add Server-Side Input Validation

Validate all fields before processing

- Email must follow standard email format
- Name must contain alphabetic characters
- Telegram handle must match pattern

Reject malformed input with HTTP 400 
Do not rely on frontend validation as it is easily bypassable

```Js
import Joi from "joi";

const schema = Joi.object({
	name: Joi.string().pattern(/^[A-Za-z\s]+$/).required(),
	email: Joi.string().email().required(),
	telegram: Joi.string().pattern(/^@[A-Za-z0-9_]{5, }+$/).required(),
	captcha: Joi.string().required()
});

app.post("/form-submit", (req, res, next) => {
  const { error } = schema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: "Invalid input: " + error.details[0].message });
  }
  next();
});

```

## Prevent automated abuse

Add bot protection measures, such as

- User-agent consistency check
- Submission timing analysis
- Request fingerprinting
Reject requests which show automated behavior

#### Realization example
```Js
if (!req.headers['user-agent'] || req.headers['user-agent'].lenght < 10)
	return res.status(400).json({error:"#Invalid User-Agent"})
```

# Validation

After applying listed fixes

- Submissions without CAPTCHA will fail
- Invalid field values will be rejected
- Rate limits should persist across recurring requests
- Automated POST requests should be blocked

# References

- CWE-307 - Improper Restriction of Excessive Requests
- CWE-20 - Improper Input Validation
- CWE-799 - Improper Control of Interaction Frequency
