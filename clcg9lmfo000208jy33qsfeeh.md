# The best time ever to learn

As I said, I have mixed feelings about ChatGPT, but one thing is for sure. There's never been a better time to learn coding, or anything IT-related. The knowledge available via ChatGPT is immense and covers anything and everything from Algorithms to designing databases or creating ssh tunnels. It even understands CSS:-)

### Algorithms

I tried quite hard, but I couldn't come up with any algorithm (sorting or more general) that ChatGPT couldn't explain AND implement. It is fair to say that ChatGPT's knowledge vastly exceeds mine in this area (well, not only in this area - to be honest..) It kept bringing stuff up I never heard of. But let's demonstrate it with some basic stuff first:

**Prompt: "Explain mergesort"**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672683901823/51474db6-35c8-4b1f-847c-a5883333f770.png align="center")

**Prompt: "Code it in JavaScript"**

```javascript
function mergeSort(array) {
  if (array.length < 2) {
    return array;
  }

  const middle = Math.floor(array.length / 2);
  const left = array.slice(0, middle);
  const right = array.slice(middle);

  return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right) {
  const result = [];

  while (left.length && right.length) {
    if (left[0] < right[0]) {
      result.push(left.shift());
    } else {
      result.push(right.shift());
    }
  }

  return result.concat(left, right);
}
/*
To use the function, you would pass in an array as an argument, and the function will return a new, sorted array. For example:
*/
const array = [5, 3, 1, 2, 4];
const sortedArray = mergeSort(array);
console.log(sortedArray); // [1, 2, 3, 4, 5]
```

Ok, maybe 'Mergesort' is too popular, how about some more 'niche' thing? How about  
[Timsort](https://en.wikipedia.org/wiki/Timsort)? By the way, I have to admit, I did not even know it existed, but this is my shortcoming.  
I tried it out, and it works.

```javascript
/*
Timsort is a complex sorting algorithm that combines elements of both comparison-based and non-comparison based sorting algorithms. It is not a simple algorithm to implement, but here is a basic implementation of Timsort in JavaScript:
*/
function timSort(array) {
  const RUN = 32;
  let n = array.length;
  let i;
  for (i = 0; i < n; i += RUN) {
    insertionSort(array, i, Math.min((i + RUN - 1), (n - 1)));
  }
  for (let size = RUN; size < n; size = 2 * size) {
    for (let left = 0; left < n; left += 2 * size) {
      let mid = left + size - 1;
      let right = Math.min((left + 2 * size - 1), (n - 1));
      merge(array, left, mid, right);
    }
  }
  return array;
}

function insertionSort(array, start, end) {
  for (let i = start + 1; i <= end; i++) {
    let temp = array[i];
    let j = i - 1;
    while (j >= start && array[j] > temp) {
      array[j + 1] = array[j];
      j--;
    }
    array[j + 1] = temp;
  }
}

function merge(array, l, m, r) {
  let n1 = m - l + 1;
  let n2 = r - m;
  let L = [];
  let R = [];
  for (let i = 0; i < n1; i++) {
    L[i] = array[l + i];
  }
  for (let j = 0; j < n2; j++) {
    R[j] = array[m + 1 + j];
  }
  let i = 0;
  let j = 0;
  let k = l;
  while (i < n1 && j < n2) {
    if (L[i] <= R[j]) {
      array[k] = L[i];
      i++;
    } else {
      array[k] = R[j];
      j++;
    }
    k++;
  }
  while (i < n1) {
    array[k] = L[i];
    i++;
    k++;
  }
  while (j < n2) {
    array[k] = R[j];
    j++;
    k++;
  }
}
```

Obviously, it is not limited to JavaScript.  
**Prompt:** Implement dept-first search in python

```python
def depth_first_search(graph, start, visited=None):
    if visited is None:
        visited = set()
    visited.add(start)
    for next in graph[start] - visited:
        depth_first_search(graph, next, visited)
    return visited

graph = {'0': set(['1', '2']),
         '1': set(['0', '3', '4']),
         '2': set(['0']),
         '3': set(['1']),
         '4': set(['2', '3'])}
depth_first_search(graph, '0')

"""
This implementation takes a graph represented as a dictionary, with the keys being the nodes and the values being a set of nodes that can be reached from the key node. The start parameter is the node to start the search from, and the visited parameter is a set of nodes that have already been visited.

The function adds the start node to the visited set and then calls itself recursively on all the nodes that can be reached from the start node and have not yet been visited. The function returns the visited set when it has finished executing.

You can run the example by copying and pasting the code into a Python interpreter. The output should be a set containing the nodes '0', '1', '3', '4', and '2'.
"""
```

I no longer see any reason why should anybody spend money on Udemy anymore. I mean, I have nothing against it, I did it myself quite a few times in the past, but I think it is over. If you want to learn Algorithms (from bubble sort to Dijkstra), ChatGPT is all you need.

### Frontend

Everything I wrote above is completely true for Frontend as well.  
**Prompt:** Create a rotating gold metal torus with lots of light in react.js

```javascript
import React, { useRef, useEffect } from 'react';
import * as THREE from 'three';

function Torus() {
  const containerRef = useRef(null);

  useEffect(() => {
    // Create a new Three.js scene
    const scene = new THREE.Scene();

    // Create a new Three.js camera
    const camera = new THREE.PerspectiveCamera(
      75, 
      window.innerWidth / window.innerHeight, 
      0.1, 
      1000
    );

    // Set the position of the camera
    camera.position.z = 5;

    // Create a new Three.js renderer
    const renderer = new THREE.WebGLRenderer({ alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);

    // Append the canvas element to the container element
    containerRef.current.appendChild(renderer.domElement);

    // Create a new Three.js torus geometry
    const geometry = new THREE.TorusGeometry(1, 0.4, 16, 100);

    // Create a new Three.js gold metal material
    const material = new THREE.MeshStandardMaterial({
      color: 0xffd700,
      roughness: 0.5,
      metalness: 1
    });

    // Create a new Three.js mesh using the geometry and material
    const torus = new THREE.Mesh(geometry, material);
    scene.add(torus);

    // Create a new Three.js point light
    const light = new THREE.PointLight(0xffffff, 1, 100);
    light.position.set(10, 10, 10);
    scene.add(light);

    // Render the scene
    renderer.render(scene, camera);

    // Animate the scene
    const animate = function () {
      requestAnimationFrame(animate);
      torus.rotation.x += 0.01;
      torus.rotation.y += 0.01;
      renderer.render(scene, camera);
    };
    animate();
  }, []);

  return <div ref={containerRef} />;
}

export default Torus;
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672743676262/b534830e-2287-49c5-b8b8-4ff3ce738873.gif align="center")

**Prompt:** Create a simple CSS flexbox example

```xml
<style>
  .container {
    display: flex;
  }
  .item {
    width: 50px;
    height: 50px;
    background-color: lightblue;
  }
</style>

<div class="container">
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
</div>

<!--
This code will create a container element that displays its child elements in a row using flexbox. The child elements (called "items" in this example) will have a width and height of 50 pixels and a light blue background color.

To use flexbox, you need to set the display property of the parent element to flex. You can then use various other flexbox properties to control the layout of the child elements, such as justify-content to control the horizontal alignment and align-items to control the vertical alignment.
-->
```

### **Database**

I am not a database expert, I never designed large databases, I just queried them with basic stuff like 'SELECT' or 'LEFT JOIN' as full-stack developers normally do. The depth, required by my daily need is very much covered.

**Prompt:** Explain "Left inner join" with an example.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672745684365/2d695af7-8e2b-4322-89c4-e368abebf54b.png align="center")

And it does not stop there, it goes on with a lengthy explanation that I don't include. But you got the idea. It does not only write the query but also explains it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673338030155/b5f30286-583d-4771-ab72-50e833b87bae.png align="center")

  
Let's see what guidance I can get if I want to design a large database.

**Prompt**: I am tasked to design a large database, which principles should I follow?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672749800698/341553a5-b1f6-41e1-97f0-ee7bb0e098b2.png align="center")

### Miscellaneous

It knows how I should set up the debugger whether in VSCode or Eclipse, and also knows how to build an ssh tunnel on ubuntu, and why would I want to do it in the first place.  
It can debug, at least small functions:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672750639873/545389b9-192a-4053-b0de-83a465a2b29c.png align="center")

To introduce a bug I changed `if(low < high){` to `if(low <= high){`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672750829504/6cd8293d-9cbb-49bf-b906-d9a013b7dd3c.png align="center")

Can it debug large, enterprise applications? No, it can't. Luckily. Because the day when it can, will be the day also when the vast majority of developers lose their jobs...