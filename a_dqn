import simpy
from config import *
from random import *
import numpy as np
import networkx as nx
import math
import utils
from collections import defaultdict
from tqdm import tqdm
import logging
from typing import List, Dict, Tuple
from queue import PriorityQueue


topology = nx.read_weighted_edgelist('topology/' + TOPOLOGY, nodetype=int)
topology = topology.to_directed()
class Desalocate(object):
    def __init__(self, env):
        self.env = env
    def Run(self, count, path, spectro, holding_time):
        global topology
        yield self.env.timeout(holding_time)
        for i in range(0, (len(path)-1)):
            for slot in range(spectro[0],spectro[1]+1):
                topology[path[i]][path[i+1]]['capacity'][slot] = 0
class Simulador(object):
    def __init__(self, env):
        self.env = env
        global topology
        for u, v in list(topology.edges):
            topology[u][v]['capacity'] = [0] * SLOTS
        self.nodes = list(topology.nodes())
        self.random = Random()
        self.NumReqBlocked = 0
        self.cont_req = 0
        self.connModulationInfo = {}
    def Run(self, rate):
        global topology
        #for count in range(1, NUM_OF_REQUESTS + 1):
        for count in tqdm(range(1, NUM_OF_REQUESTS+1), desc="Processing Requests"):
            yield self.env.timeout(self.random.expovariate(rate))
            src, dst = self.random.sample(self.nodes, 2)
            bandwidth = int(self.random.uniform(1, 200))
            holding_time = self.random.expovariate(HOLDING_TIME)

            # TODO: Use A* with DQN to find the k_paths
            paths = self.find_k_paths(src, dst, N_PATH)
            flag = 0
            #print("paths",paths)
            
            for path in paths:
                distance = int(utils.Distance(topology, path))
                #print("distance",distance)
                if distance <= 4000:
                    num_slots, m = utils.Modulation(distance, bandwidth)
                    self.check_path = utils.PathIsAble(num_slots, path, topology)
                    if self.check_path[0] == True:
                        OSNR_r_path = utils.computeOSNR(topology, path, num_slots, self.check_path[1], m, self.connModulationInfo)
                        #print("m", m, "OSNR_r_path",OSNR_r_path)
                        check_OSNR_Th= utils.check_OSNR_Th(m, OSNR_r_path)
                        if check_OSNR_Th == True:
                            self.cont_req += 1
                            utils.FirstFit(topology,count, self.check_path[1],self.check_path[2],path)
                            self.connModulationInfo[count] = m
                            spectro = [self.check_path[1], self.check_path[2]]
                            desalocate = Desalocate(self.env)
                            self.env.process(desalocate.Run(count,path,spectro,holding_time))
                            flag=1
                            break
                       
                    
            if flag == 0:
                self.NumReqBlocked += 1
