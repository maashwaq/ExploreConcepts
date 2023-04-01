**Java using the Apache ZooKeeper Curator framework**

import org.apache.curator.framework.CuratorFramework;

import org.apache.curator.framework.CuratorFrameworkFactory;

import org.apache.curator.retry.ExponentialBackoffRetry;

import org.apache.curator.framework.recipes.cache.NodeCache;

import org.apache.curator.framework.recipes.cache.NodeCacheListener;

import org.apache.zookeeper.CreateMode;

import org.apache.zookeeper.data.Stat;

public class CuratorExample {
    
    private static final String ZK_ADDRESS = "localhost:2181";
    private static final String ZK_PATH = "/example/path";
    
    public static void main(String[] args) throws Exception {
        
        // Create CuratorFramework client
        CuratorFramework client = CuratorFrameworkFactory.newClient(ZK_ADDRESS, new ExponentialBackoffRetry(1000, 3));
        client.start();
        
        // Create a node in ZooKeeper
        client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath(ZK_PATH, "exampleData".getBytes());
        
        // Read the data of the created node
        Stat stat = new Stat();
        byte[] data = client.getData().storingStatIn(stat).forPath(ZK_PATH);
        System.out.println("Data: " + new String(data));
        
        // Watch the node for changes
        NodeCache nodeCache = new NodeCache(client, ZK_PATH);
        nodeCache.start(true);
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                byte[] data = nodeCache.getCurrentData().getData();
                System.out.println("Data changed: " + new String(data));
            }
        });
        
        // Update the data of the node
        client.setData().withVersion(stat.getVersion()).forPath(ZK_PATH, "newData".getBytes());
        
        // Wait for node change event
        Thread.sleep(1000);
        
        // Close the client
        client.close();
    }
}

In this example, the CuratorFramework client is created and started with the localhost:2181 address of the ZooKeeper ensemble, and an ExponentialBackoffRetry retry policy. A node is created in ZooKeeper with the /example/path path and exampleData data.

The data of the created node is then read and printed to the console. 
A NodeCache is created to watch the node for changes, and a NodeCacheListener is added to print the updated data to the console 
when the node is changed. The data of the node is then updated to newData.

After waiting for the node change event for one second, the client is closed.

