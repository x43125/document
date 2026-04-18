# 跳表(SkipList)

> Java实现



```java
package Java.RedisStructure;

import java.util.Random;

/**
 * SkipList
 */
public class SkipList {
    /**
     * Skip Node inner Class
     */
    class SkipNode {
        int val;
        SkipNode[] next;

        public SkipNode(int MAX_LEVEL, int val) {
            this.val = val;
            /**
             * node array of now level
             */
            this.next = new SkipNode[MAX_LEVEL];
        }
    }

    /**
     * max level of skip node
     */
    private static final int MAX_LEVEL = 32;
    /**
     * nov level of skip node
     */
    private int level = 0;
    /**
     * header skip node
     */
    private final SkipNode HEADER = new SkipNode(MAX_LEVEL, -1);
    /**
     * random manager
     */
    private final Random RANDOM = new Random();
    /**
     * environment number
     */
    private final double E = Math.E;

    /**
     * find method:
     * 
     * @param val the value to find
     * @return whether the result was found: found is true
     */
    public boolean contains(int val) {
        // search from the header node
        SkipNode cur = HEADER;
        // search from top level to the bottom
        for (int i = level; i >= 0; i--) {
            // search now level from header to last, when node.val less than next
            while (cur.next != null && cur.next[i].val < val) {
                cur = cur.next[i];
            }
            // if node.val equals val return true
            if (cur.next[0].val == val) {
                return true;
            }
        }
        // if search ends and nothing is found, return false
        return false;
    }

    /**
     * insert method
     * 
     * @param val new node's value
     */
    public void insert(int val) {
        // put cur on the head node
        SkipNode cur = HEADER;
        // to store preceding nodes
        SkipNode[] predecessors = new SkipNode[MAX_LEVEL];
        // traverse all level to find all preceding nodes
        for (int i = level; i >= 0; i--) {
            cur = HEADER;
            while (cur.next[i] != null && cur.next[i].val < val) {
                cur = cur.next[i];
            }
            predecessors[i] = cur;
        }
        // make cur to next node
        cur = cur.next[0];
        // determine the node's level
        int nextLevel = randomLevel();
        // insert new node
        if (cur == null || cur.val != val) {
            if (nextLevel > level) {
                predecessors[nextLevel] = HEADER;
                level = nextLevel;
            }
            // insert new node into list
            SkipNode node = new SkipNode(MAX_LEVEL, val);
            for (int i = level; i >= 0; i--) {
                node.next[i] = predecessors[i].next[i];
                predecessors[i].next[i] = node;
            }
        }
    }

    /**
     * delete method
     * 
     * @param val value to delete
     */
    public void delete(int val) {
        SkipNode cur = HEADER;
        SkipNode[] predecessors = new SkipNode[MAX_LEVEL];

        // switch to delete node
        for (int i = level; i >= 0; i--) {
            cur = HEADER;
            while (cur.next != null && cur.next[0].val < val) {
                cur = cur.next[0];
            }
            predecessors[i] = cur;
        }
        // next node is the node to delete
        cur = cur.next[0];
        if (cur.val != val) {
            return;
        }
        for (int i = level; i >= 0; i--) {
            if (predecessors[i].next[i].val != val) {
                continue;
            }
            // delete node
            predecessors[i].next[i] = cur.next[i];
        }
        while (level > 0 && HEADER.next[level] == null) {
            level--;
        }
    }

    /**
     * skip list string
     */
    public String toString() {
        StringBuilder sb = new StringBuilder();
        SkipNode cur = HEADER.next[0];
        // SkipNode cur = HEADER;
        sb.append("{");
        while (cur.next[0] != null) {
            sb.append(cur.val).append(",");
            cur = cur.next[0];
        }
        sb.append(cur.val).append("}");
        return sb.toString();
    }

    /**
     * random level generate manager
     * 
     * @return
     */
    private int randomLevel() {
        double ins = RANDOM.nextDouble();
        int nextLevel = level;
        if (ins > E && level < MAX_LEVEL) {
            nextLevel++;
        }
        return nextLevel;
    }

    public static void main(String[] args) {
        SkipList skipList = new SkipList();
        skipList.insert(0);
        skipList.insert(1);
        skipList.insert(2);
        skipList.insert(3);
        skipList.insert(4);
        skipList.insert(6);
        skipList.insert(7);
        skipList.insert(8);
        skipList.insert(9);
        skipList.insert(10);
        skipList.insert(11);
        skipList.insert(12);
        skipList.insert(13);
        skipList.insert(15);

        System.out.println("skip node list: " + skipList.toString());
        System.out.println("skip list contains 13: " + skipList.contains(13));
        System.out.println("skip list contains 14: " + skipList.contains(14));
        System.out.println("skip list contains 5: " + skipList.contains(5));
        System.out.println("insert 5");
        skipList.insert(5);
        System.out.println("skip node list: " + skipList.toString());
        System.out.println("skip list contains 5: " + skipList.contains(5));
        System.out.println("delete node 5");
        skipList.delete(5);
        System.out.println("skip node list: " + skipList.toString());
        System.out.println("skip list contains 5: " + skipList.contains(5));
    }
}
```

