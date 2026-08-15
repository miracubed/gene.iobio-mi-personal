gene.iobio-mi-personal Startup Script Creation  

# Make Backend Startup Script ~/start_network_backend.sh  

## Step 1  
bash  
```  
nano ~/start_network_backend.sh
```  

## Step 2  
Copy and Paste (right click or Edit > Paste or whatever method works on your system) into the nano file you just opened:  
```  
#!/bin/bash

echo "Starting gene.iobio gru-backend-2.0.0..."
cd ~
podman start gru-backend-2.0.0
sleep 5
curl -s localhost:9001

echo "Backend started."
echo "Okay to start frontend."
echo ""
echo "In a different WSL window type:"
echo "~/start_network_frontend.sh"
echo "CTRL+C in frontend server window to stop it."
echo ""
echo "Stop the backend server by typing:"
echo "podman stop gru-backend-2.0.0"
```

## Step 3  
Save it: CTRL+O, ENTER, CTRL+X  

View it  
bash  
```  
cat ~/start_network_backend.sh
```  
## Step 4  
Make it executable  
bash  
```  
chmod -x ~/start_network_backend.sh
```  
===  

# Make Frontend Startup Script ~/start_network_frontend.sh  

## Step 1  
bash  
```  
nano ~/start_network_frontend.sh
```  

## Step 2  
Copy and Paste (right click) into the nano file you just opened:  
```  
#!/bin/bash

echo "Starting gene.iobio frontend on network interface..."
cd ~/gene.iobio/client

# Verify backend is running
if ! curl -s http://localhost:9001 > /dev/null; then
  echo "ERROR: gru-backend not running on port 9001"
  exit 1
fi

echo "Backend confirmed running on localhost:9001"
echo "Starting frontend on 0.0.0.0:3000..."
echo ""
echo "Access from local browser: http://localhost:3000"
echo "Access from network (.52): http://192.168.1.240:3000"
echo ""

serve -s . -l 3000
```  
View it  
bash  
```  
cat ~/start_network_frontend.sh
```  
## Step 4  
Make it executable  
bash  
```  
chmod -x ~/start_network_frontend.sh
```  

---

≡≡≡≡≡ END: E_Extra_Startup_Scripts.md ≡≡≡≡≡  
