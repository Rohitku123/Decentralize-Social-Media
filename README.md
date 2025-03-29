# Decentralize-Social-Media
Programming Code for it

import React, { useEffect, useState } from "react";
import Web3 from "web3";
import SocialMediaContract from "./contracts/SocialMedia.json";

const App = () => {
  const [account, setAccount] = useState("");
  const [contract, setContract] = useState(null);
  const [posts, setPosts] = useState([]);
  const [newPost, setNewPost] = useState("");

  useEffect(() => {
    const init = async () => {
      if (window.ethereum) {
        const web3 = new Web3(window.ethereum);
        await window.ethereum.request({ method: "eth_requestAccounts" });
        const accounts = await web3.eth.getAccounts();
        setAccount(accounts[0]);

        const networkId = await web3.eth.net.getId();
        const deployedNetwork = SocialMediaContract.networks[networkId];
        if (deployedNetwork) {
          const instance = new web3.eth.Contract(
            SocialMediaContract.abi,
            deployedNetwork.address
          );
          setContract(instance);
        }
      }
    };
    init();
  }, []);

  const createPost = async () => {
    if (contract && newPost) {
      await contract.methods.createPost(newPost).send({ from: account });
      setNewPost("");
      loadPosts();
    }
  };

  const loadPosts = async () => {
    if (contract) {
      const postCount = await contract.methods.getPostCount().call();
      const loadedPosts = [];
      for (let i = 0; i < postCount; i++) {
        const post = await contract.methods.posts(i).call();
        loadedPosts.push(post);
      }
      setPosts(loadedPosts);
    }
  };

  useEffect(() => {
    if (contract) {
      loadPosts();
    }
  }, [contract]);

  return (
    <div>
      <h1>Decentralized Social Media</h1>
      <p>Connected Account: {account}</p>
      <input
        type="text"
        value={newPost}
        onChange={(e) => setNewPost(e.target.value)}
        placeholder="Write a post..."
      />
      <button onClick={createPost}>Post</button>
      <h2>Posts</h2>
      <ul>
        {posts.map((post, index) => (
          <li key={index}>{post.content}</li>
        ))}
      </ul>
    </div>
  );
};

export default App;
