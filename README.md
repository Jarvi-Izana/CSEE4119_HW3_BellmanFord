# 1,DESCRIPTION OF THE CODE
## class neighbor:
### MAIN VARIABLE
link_status: *boolean* decide whether the target router is open.
link_down: *boolean* decide whether the link from host to neighbor router.
weight: *float* cost to the dest router, will be set INF when the corresponding link is down
origin: *float* origin cost to neighbor, won't change, used for link up to restore the cost.

## class DV:
### MAIN VARIABLE
__ceiling: *float*, to check whether the distance is infinite.__
neighbors: *dict*, k-v: hostname-neighbor_instance.
distance_vector: *dict:dict*, k-v: hostname- *dict{hostname:distance}*
timeout: *float*, used to send dv and check whether its neighbor is open.
dv_update_flag: *boolean*, if True, broadcast dv to neighbors still open and linked in.

### MAIN METHOD
update_dv: update certain entry of the distance vector if the corresponding dest is in link __(not link down)__ and open. 
recv: recv msg from the socket and invoke corresponding method to invoke.
send: if timeout or dv updated, it will send the dv of the host to all its neighbors open and in link.
timer_: one timeout time to broadcast dv if the dv is unchanged. 3 timeout to decide whether certain neighbor is alive.
link_up: restore certain link to dest router.
link_down: link down the link to certain dest.
bellman_ford: __core__ method to realize the *Bellman ford* algorithm. If the dv of the host is changed, if will set the
dv_status to True so that the *send* method will broadcast the host dv.
df_client: initialize all the infrastructure. Allocate 3 threads to recv(), send(), timer_() respectively. Receive all the 
commands from the console.

# 2,DETAILS AND DEVELOPMENT ENVIRONMENT
* MAC OS X
* Python 2.7.10

# 3,INSTRUCTION ON HOW TO RUN YOUR CODE
python bfclient.py <host_ip> <host_port> <timeout> __<dest_ip> <dest_port> <host2dest_distance>__
THERE CAN BE AS MANY AS DESTINATIONS AS YOU WANT, AND MUST IN FORMAT OF TUPLE 0F 3 LIKE THIS:
__<dest_ip> <dest_port> <host2dest_distance>__

# 4,SIMPLE COMMANDS TO INVOKE YOUR CODE
## INIT THE TOPOLOGICAL STRUCTURE.
__IF YOU LINK DOWN CERTAIN LINK, YOU MUST USE LINK UP TO RESTORE THE LINK, OTHERWISE THE STATUS OF LINK WILL ALWAYS
BE *FALSE* EVEN WHEN YOU RESTART CORRESPONDING DEST. WHEN YOU RESTART THE DEST, THE DEST DOESN'T KNOW THE LINK IS DOWN.
AND WILL THINK THE LINK IS VALID. BUT THE HOST WILL NOT RESTORE THE DEST EVEN THOUGH IT RECEIVES DV FORM DEST, INDICATING THE 
 LINK IS OPEN AGAIN. EVENTUALLY, DEST WILL SET THE LINK_STATUS TO FALSE AFTER 3 TIMEOUTS.__
YOU MAY WAIT FOR SOME TIME TO LET THE DISTANCE VECTOR TO CONVERGE WHEN YOU LINK DOWN A LINK.
JUST USE *SHOWRT* TO SEE WHETHER THE DISTANCE IS GROWING.
THE DISTANCE WILL GROW RAPIDLY UNTIL IT REACHES THE CEILING. ONCE THE DISTANCE EXCEEDS THE 
CEILING VALUE, THE DISTANCE BETWEEN THESE TWO ROUTER WILL BE SET INFINITE. AND DISTANCE TO 
THIS ROUTER WILL NOT SHOW UP WHEN YOU LOOK UP THE ROUTING TABLE.
IF NOT GROWING, IT CONVERGES ALREADY.

## TRIANGLE:
python bfclient.py 127.0.0.1 4115 3 127.0.0.1 4119 6.0 127.0.0.1 4118 3.0
python bfclient.py 127.0.0.1 4119 3 127.0.0.1 4115 6.0 127.0.0.1 4118 2.0
python bfclient.py 127.0.0.1 4118 3 127.0.0.1 4115 3.0 127.0.0.1 4119 2.0

127.0.0.1 4119>>> LINKDOWN 127.0.0.1 4118
127.0.0.1 4119>>> SHOWRT
127.0.0.1 4119>>> LINKUP 127.0.0.1 4118
127.0.0.1 4119>>> SHOWRT
127.0.0.1 4119>>> CLOSE
127.0.0.1 4119>>> SHOWRT
127.0.0.1 4119>>> TA

## MORE SOPHISTICATED:
python bfclient.py 127.0.0.1 4115 3 127.0.0.1 4119 6.0 127.0.0.1 4118 3.0 127.0.0.1 4116 5.0
python bfclient.py 127.0.0.1 4119 3 127.0.0.1 4115 6.0 127.0.0.1 4118 2.0
python bfclient.py 127.0.0.1 4118 3 127.0.0.1 4115 3.0 127.0.0.1 4119 2.0 127.0.0.1 4116 4.0
python bfclient.py 127.0.0.1 4116 3 127.0.0.1 4118 4.0 127.0.0.1 4115 5.0

127.0.0.1 4116>>> LINKDOWN 127.0.0.1 4118
127.0.0.1 4116>>> SHOWRT
127.0.0.1 4116>>> LINKUP 127.0.0.1 4118
127.0.0.1 4116>>> SHOWRT
127.0.0.1 4116>>> CLOSE
127.0.0.1 4116>>> SHOWRT
127.0.0.1 4119>>> TA

# 5,DESCRIPTION ADDITIONAL FUNCTION AND HOW THEY CAN BE TESTED
1. SHOW FULL TABLE IN DETAIL:
*COMMAND 'TA'*
SHOW THE ROUTING TABLE INCLUDING ALL NEIGHBORS OPEN OR CLOSED.
ALSO WITH THE NEXT HOP.

2. ceiling:
*CMD: MAG 15*
WILL SET THE CEILING MAGNITUDE TO BE 15
WHEN YOU LAUNCH THE HOST, YOU WILL SEE THE PROMPT:
'The ceiling is: 90.0'
IF DISTANCE EXCEEDS THE ceiling, THE DISTANCE BETWEEN WILL BE SET INFINITE, AND THE ROUTER WON'T SHOW UP WHEN YOU USE
COMMAND *SHOWRT*. THE ceiling IS COMPUTED AS FORMULA BELLOW:
max{cost to neighbor} * mag. The default mag is 15.

3. MODIFIED *SHOWRT*
IF THE DISTANCE IS INFINITE, MEANING THERE'S NO LINK BETWEEN, THE TABLE WILL NOT SHOW THE DISTANCE TO SUCH DEST.