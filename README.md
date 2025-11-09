# SPL-Token
How to make your own SPL-Token on the Solana Devnet.



Linux/Windows (WSL)
https://docs.docker.com/engine/install/ (link for all other distros)
If you are using Ubuntu (primary OS or WSL), do this:
here is the link to documentation: https://docs.docker.com/engine/install/ubuntu/


# Setup the Docker's apt repo
<code>sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update</code>




# install the latest Docker version

<code>sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin</code>


<b>Now make sure it works correctly</b>

<code>sudo docker run hello-world</code>

<b>Setup Solana Docker Container</b>

<b>now create a new folder for your Token-Project</b>

<code>mkdir your-token-name</code>
      now go to the directory
<code>cd your-token-name</code>




# now create a Dockerfile

##we use nano to create a file
<code>nano Dockerfile</code>
##copy and paste the Dockerfile code down below, use ctrl+x+enter to save.





# Dockerfile-Code


<code># Use a lightweight base image
FROM debian:bullseye-slim
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    curl build-essential libssl-dev pkg-config nano \
    && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
ENV PATH="/root/.cargo/bin:$PATH"
RUN rustc --version
RUN curl -sSfL https://release.anza.xyz/stable/install | sh \
    && echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc
ENV PATH="/root/.local/share/solana/install/active_release/bin:$PATH
RUN solana --version
RUN solana config set -ud
WORKDIR /solana-token
CMD ["/bin/bash"]</code>



# Build the Docker Image

Now were building our Docker Image with the Dockerfile which we just created.  The name we give the image is heysolana.

<code>docker build -t heysolana .</code>


# Run the container 
now we create a docker container with this image (and running it). Using the -it switch will throw us right into the container.
-v options are mapping our current directory inside the docker container, that all the work we do will be saved.
We are naming the Docker container heysolana.

<code>docker run -it --rm -v $(pwd):/solana-token -v $(pwd)/solana-data:/root/.config/solana heysolana</code>

# Create a Solana Token on Devnet

Why are we creating it on the Devnet?
Because it’s free, and this might be where you will stop. You can do everything the mainnet can do, it is just considered a testing environment and you can’t buy or sell tokens here. 
BUT…you can send and receive tokens. 
So, if that’s all you care about, this is where you can leave off.

# Create an Account for Mint-Authority

This will be the account that will own the token we’re creating.
This will also be a Solana wallet. You can send and receive Solana tokens with this wallet.
<code>solana-keygen grind --starts-with dra:1</code>


# Set the Account as default Keypair

With this command, we are telling the Solana-CLI that it should use this account at whatever we do.
<code>solana config set --keypair dra-your-token-acount.json</code>

# Change to devnet
With this command we are moving to the devnet for creating the Token.
<code>solana config set --url devnet</code>

# Get some Sol from a faucet

In order to mint new tokens, send tokens etc.on the Solana network, you’ll need SOL for Transaction fees.
On the mainnet you would need to buy actual SOL. 
On the devnet, you can just get an airdrop.
Go out to this URL (https://solfaucet.com/)
To get SOL airdropped to the account you just created. 
What’s your account address?

<code>## Run this to find your solana address
solana address</code>

Paste your solana address in the field and enter an amount of SOL you want.

# Create a Mint address

Yes, we create another Account.
This will be your mint address, the factory that will make your token.

<code>solana-keygen grind --starts-with mnt:1</code>


# Now let's mint our Token!

This is using the default token program from Solana with the –progam-id switch.
We’re enabling metadata with –enable-metadata, you know, pictures and stuff.
Make sure to include your own mint address below.

<code>spl-token create-token \
--program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb \
--enable-metadata \
--decimals 9 \
mnt-your-mint-address.json</code>


# Upload Metadata

Now, we are going to upload the metadata of our token, things like the name, symbol and the icon.
Let’s start with our token icon. Make sure it is:
Square
either 512x512 or 1024x1024
less than 100kb

![CompressJPEG Online_img(512x512)](https://github.com/user-attachments/assets/22fd8582-5a08-4c6a-a8cc-ed4eec9540d0)

# Upload the image to some sort of online-storage

It would be nice to store our stuff on decentralized storage. 
I tried Pinata and I like it.

—> https://app.pinata.cloud/
Create an account and upload your image.

1. Go to Files at IPFS
  <img width="360" height="243" alt="image" src="https://github.com/user-attachments/assets/95a1f2d3-8105-4e29-9164-050249372007" />

2. click add and upload your picture.
   <img width="227" height="172" alt="image" src="https://github.com/user-attachments/assets/1c50bedf-8e83-4635-8100-0040f18fa5a5" />

   Open your file and copy the URL. It should look something like this. Put it somewhere, we’re about to use it.
   
   <code>https://aquamarine-bitter-condor-374.mypinata.cloud/ipfs/bafkreibznjh4d5bnhgbtbqcuwq5ccfyy533lri4w2zvigcdhlmd2aa3f4y</code>
   

# Create a metadata File
Create a file with this format, filling in your details.
Name can be anything you want
Symbol is the short version, like a stock symbol.
The image will be the url we just copied from our image upload process. 
f you don’t have one, feel free to use mine.


 <b>Save this as a json file</b>

   <code>##create the file
         nano metadata.json
         ## Copy and paste the metadata stuff</code>


<code>{
  "name": "Drav Web3",
  "symbol": "Drav",
  "description": "This SPL-Token was created by Drav.",
  "image": "https://salmon-effective-mosquito-301.mypinata.cloud/ipfs/bafkreigaiqsulww3fulikwx5ho2ifpfqfnco5bhqs4ui5gk5rn4ggkjuoy"
}</code>

# Upload this Metadata file to Pinata
Same steps as before. Upload to Pinata and copy the url.

<img width="1212" height="236" alt="image" src="https://github.com/user-attachments/assets/49b14c1c-b853-4182-a5a7-8c2f19243ea1" />

# Add the metadata to the Token
Now it's time, to upload our metadata. This will actually go on chain.
Notice our token address is given without the .json
Then we specify our coin name,
And then our symbol.
I know, this is probably redundant.
But we’re making sure the job gets done.
And finally, the url to our metadata.

<code>spl-token initialize-metadata \
mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC \
"Drav Web3" \
"Drav" \
https://aquamarine-bitter-condor-374.mypinata.cloud/ipfs/bafkreicuwdt5fcsmciyvrn7hbes7anogeikdbamfuqo2hyeg7h4x7s3qdi</code>



# Let's take a look!

Go out to Solana Explorer (https://explorer.solana.com/), select the devnet as your cluster.

<img width="1698" height="814" alt="image" src="https://github.com/user-attachments/assets/98ef170e-f72c-440f-9bd5-53eb1cd0542d" />

Paste your token address in the search bar and CELEBRATE!!

# Let's make some Tokens!

<b>Create a Token Account</b>
please use your own Mint Address (mnt)
<code>spl-token create-account mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC</b> 

# Let's mint them!

With our token account created, let’s mint some tokens. 
We can make as many as we want and we can do this multiple times 
(as many as you would like….you’re the government now!)
We'll start with 1000.

<code>spl-token-mint mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC 1000</code>


Let's check the balance of our wallet (use your mnt)

<code>spl-token balance mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC</code>






# Going to market


<code>spl-token authorize mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC mint --disable</code>



<code>spl-token authorize mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC freeze --disable</code>


if you ever need to update your metadata


<code>spl-token update-metadata mntLd4Awni95eZTLTsbayLPNNF7xM9vtrGXxDrTXcSC uri https://salmon-effective-mosquito-301.mypinata.cloud/ipfs/QmZX81uxJvhtoaAwrcqSdcsbJz1pWZv4gJje17bYTz3Fo3</code>






# Thank you, if you need help or want to ask something, feel free to contact me on Instagram 
https://instagram.com/web3drav
