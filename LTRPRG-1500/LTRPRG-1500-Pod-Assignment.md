# Network Automation with Ansible Pod Assignment

# Lab Nodes Login Credentials
- username: cisco
- password: cisco
- Only login to the Ansible-Controller Node

# User pod assignment

| User | Pod | Ansible Controller | IOS Router | XR Router|
|------|-----|--------------------|------------|-----------|
| GOODACRE, TIM |	Pod 3 | 172.16.101.127 | 172.16.101.125 | 172.16.101.126 |
| SHIBESHI, ERMIAS |	Pod 4 | 172.16.101.170 | 172.16.101.166 | 172.16.101.17 |
| HANSON, TONY  |	Pod 5 | 172.16.101.141 | 172.16.101.14 | 172.16.101.140 |
| GU, SUSAN |	Pod 6 | 172.16.101.173 | 172.16.101.171 | 172.16.101.172 |
| LINARES, REYNALDO |	Pod 7 | 172.16.101.13 | 172.16.101.12 | 172.16.101.128 |
| CARRION, JUAN J |	Pod 8 | 172.16.101.135 | 172.16.101.133 | 172.16.101.134 |
| Rod Ethridge |	Pod 10 | 172.16.101.147 | 172.16.101.145 | 172.16.101.146 |
| SUNMONU, KENNY |	Pod 11 | 172.16.101.151 | 172.16.101.149 | 172.16.101.150 |
| Rene Agana |	Pod 12 | 172.16.101.139 | 172.16.101.137 | 172.16.101.138 |
| ------ |	Pod 13 | 172.16.101.165 | 172.16.101.163 | 172.16.101.164 |
| ------ |	Pod 14 | 172.16.101.16 | 172.16.101.152 | 172.16.101.159 |
| ------ |	Pod 15 | 172.16.101.148 | 172.16.101.143 | 172.16.101.144 |
| ------ |	Pod 16 | 172.16.101.169 | 172.16.101.167 | 172.16.101.168 |
| ------ |	Pod 17 | 172.16.101.177 | 172.16.101.175 | 172.16.101.176 |
| ------ |	Pod 18 | 172.16.101.162 | 172.16.101.160 | 172.16.101.161 |


|	User	|	Pod	|	Ansible	Controller	|	IOS	Router	|	XR	Router|
|------|-----|--------------------|------------|-----------|												
|	Kiran	Bondalakunta	|	Pod	3	|	ansible_host=172.16.101.205	|	ansible_host=172.16.101.203	|	ansible_host=172.16.101.204	|
|	Sachin	Danait	|	Pod	4	|	ansible_host=172.16.102.212	|	ansible_host=172.16.102.218	|	ansible_host=172.16.102.211	|
|	Vishesh	Kansal	|	Pod	5	|	ansible_host=172.16.101.215	|	ansible_host=172.16.101.213	|	ansible_host=172.16.101.214	|
|	Carlos	Ramirez	|	Pod	6	|	ansible_host=172.16.101.133	|	ansible_host=172.16.101.131	|	ansible_host=172.16.101.132	|
|	Phillip	Boulo	|	Pod	7	|	ansible_host=172.16.101.126	|	ansible_host=172.16.101.123	|	ansible_host=172.16.101.125	|
|	Nick	Khemani	|	Pod	8	|	ansible_host=172.16.101.107	|	ansible_host=172.16.101.124	|	ansible_host=172.16.101.103	|
|	Ash	Sisalem	|	Pod	10	|	ansible_host=172.16.101.183	|	ansible_host=172.16.101.181	|	ansible_host=172.16.101.182	|
|	KASHIF	QURESHI	|	Pod	11	|	ansible_host=172.16.102.227	|	ansible_host=172.16.102.233	|	ansible_host=172.16.102.226	|
|	Neetika	Mittal	|	Pod	12	|	ansible_host=172.16.102.200	|	ansible_host=172.16.102.204	|	ansible_host=172.16.102.199	|
|	Scott	Hereld	|	Pod	13	|	ansible_host=172.16.101.211	|	ansible_host=172.16.101.21	|	ansible_host=172.16.101.210	|
|	Venu	Narra	|	Pod	14	|	ansible_host=172.16.101.106	|	ansible_host=172.16.101.104	|	ansible_host=172.16.101.105	|
|	Sumit		|	Pod	15	|	ansible_host=172.16.101.102	|	ansible_host=172.16.101.100	|	ansible_host=172.16.101.101	|
|	Les	Hitchcock	|	Pod	16	|	ansible_host=172.16.101.198	|	ansible_host=172.16.101.196	|	ansible_host=172.16.101.197	|
|	Darren	Powel	|	Pod	17	|	ansible_host=172.16.101.118	|	ansible_host=172.16.101.116	|	ansible_host=172.16.101.117	|
|	Ray	Leung	|	Pod	18	|	ansible_host=172.16.101.236	|	ansible_host=172.16.101.234	|	ansible_host=172.16.101.235	|
|	Mark	Juza	|	Pod	19	|	ansible_host=172.16.102.231	|	ansible_host=172.16.102.23	|	ansible_host=172.16.102.230	|
|	Francisco	Arias	|	Pod	20	|	ansible_host=172.16.101.172	|	ansible_host=172.16.101.170	|	ansible_host=172.16.101.171	|
|	Brad	Zellefrow	|	Pod	21	|	ansible_host=172.16.101.150	|	ansible_host=172.16.101.149	|	ansible_host=172.16.101.15	|
|	Carlos	Pinzon	|	Pod	22	|	ansible_host=172.16.101.137	|	ansible_host=172.16.101.135	|	ansible_host=172.16.101.136	|
|	Luis	Kopti	|	Pod	23	|	ansible_host=172.16.101.13	|	ansible_host=172.16.101.128	|	ansible_host=172.16.101.129	|
|	TK	Periasamy	|	Pod	24	|	ansible_host=172.16.102.198	|	ansible_host=172.16.102.213	|	ansible_host=172.16.102.196	|
|	Yossi	Meloch	|	Pod	25	|	ansible_host=172.16.101.147	|	ansible_host=172.16.101.145	|	ansible_host=172.16.101.146	|
|	swapna	kuppani	|	Pod	26	|	ansible_host=172.16.102.216	|	ansible_host=172.16.102.229	|	ansible_host=172.16.102.215	|
|	Satish	Pilla	|	Pod	27	|	ansible_host=172.16.101.187	|	ansible_host=172.16.101.185	|	ansible_host=172.16.101.186	|
|	Helio		|	Pod	28	|	ansible_host=172.16.101.140	|	ansible_host=172.16.101.139	|	ansible_host=172.16.101.14	|
|	Noah	Soprala	|	Pod	29	|	ansible_host=172.16.101.165	|	ansible_host=172.16.101.163	|	ansible_host=172.16.101.164	|
|	Sujith	Subhash	|	Pod	30	|	ansible_host=172.16.101.209	|	ansible_host=172.16.101.207	|	ansible_host=172.16.101.208	|
|	Nadeem	Parvez	|	Pod	31	|	ansible_host=172.16.102.203	|	ansible_host=172.16.102.209	|	ansible_host=172.16.102.202	|
|	Sowjanya	Yellaboina	|	Pod	32	|	ansible_host=172.16.102.221	|	ansible_host=172.16.102.219	|	ansible_host=172.16.102.220	|
|	Dan	Kyburz	|	Pod	33	|	ansible_host=172.16.101.239	|	ansible_host=172.16.101.237	|	ansible_host=172.16.101.238	|
|	Madhavi	Nannapaneni	|	Pod	34	|	ansible_host=172.16.101.122	|	ansible_host=172.16.101.120	|	ansible_host=172.16.101.121	|
|	Dafne	Mendoza	|	Pod	35	|	ansible_host=172.16.101.119	|	ansible_host=172.16.101.111	|	ansible_host=172.16.101.115	|
|	Archana	Singh	|	Pod	36	|	ansible_host=172.16.102.21	|	ansible_host=172.16.102.207	|	ansible_host=172.16.102.208	|
|	Amy	Kalinski	|	Pod	37	|	ansible_host=172.16.101.225	|	ansible_host=172.16.101.223	|	ansible_host=172.16.101.224	|
|	Sutinder	Sethi	|	Pod	38	|	ansible_host=172.16.101.158	|	ansible_host=172.16.101.156	|	ansible_host=172.16.101.157	|
|	Janmesh	Jani	|	Pod	39	|	ansible_host=172.16.102.193	|	ansible_host=172.16.102.197	|	ansible_host=172.16.102.192	|
|	Manish	Jani	|	Pod	40	|	ansible_host=172.16.102.224	|	ansible_host=172.16.102.234	|	ansible_host=172.16.102.223	|
|	Heine	Melo	|	Pod	41	|	ansible_host=172.16.101.94	|	ansible_host=172.16.101.92	|	ansible_host=172.16.101.93	|
|	Corey	Jackson	|	Pod	42	|	ansible_host=172.16.101.194	|	ansible_host=172.16.101.192	|	ansible_host=172.16.101.193	|
|	Diego	Castro-Zumaeta	|	Pod	43	|	ansible_host=172.16.101.114	|	ansible_host=172.16.101.112	|	ansible_host=172.16.101.113	|
|	Scott	Stauffer	|	Pod	44	|	ansible_host=172.16.101.30	|	ansible_host=172.16.101.28	|	ansible_host=172.16.101.29	|
