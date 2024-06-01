const solanaWeb3 = require('@solana/web3.js');

// Function to establish connection to the Devnet
const establishConnection = () => {
    return new solanaWeb3.Connection(solanaWeb3.clusterApiUrl('devnet'), 'confirmed');
};

// Function to create a new account
const createNewAccount = () => {
    return solanaWeb3.Keypair.generate();
};

// Function to transfer lamports
const transferLamports = async (connection, from, toPublicKey, lamports) => {
    const transaction = new solanaWeb3.Transaction().add(
        solanaWeb3.SystemProgram.transfer({
            fromPubkey: from.publicKey,
            toPubkey: toPublicKey,
            lamports: lamports,
        })
    );
    const signature = await solanaWeb3.sendAndConfirmTransaction(connection, transaction, [from]);
    return signature;
};

// Main function to execute tasks
const main = async () => {
    const connection = establishConnection();
    const feePayer = createNewAccount();

    console.log('Requesting Airdrop of 2 SOL for fee payer...');
    const airdropSignature = await connection.requestAirdrop(feePayer.publicKey, 2 * solanaWeb3.LAMPORTS_PER_SOL);
    await connection.confirmTransaction(airdropSignature);

    // Task 1: Create a new account
    const newAccount1 = createNewAccount();
    console.log('New Account 1:', newAccount1.publicKey.toBase58());

    // Task 2: Transfer 5,000 lamports to a specified account
    const specifiedAccount = new solanaWeb3.PublicKey('63EEC9FfGyksm7PkVC6z8uAmqozbQcTzbkWJNsgqjkFs');
    const txSignature2 = await transferLamports(connection, feePayer, specifiedAccount, 5000);
    console.log('Transfer 5,000 lamports to specified account:', txSignature2);

    // Task 3: Create a new account and transfer 5,000 lamports to it
    const newAccount2 = createNewAccount();
    console.log('New Account 2:', newAccount2.publicKey.toBase58());
    const txSignature3 = await transferLamports(connection, feePayer, newAccount2.publicKey, 5000);
    console.log('Create new account and transfer 5,000 lamports to it:', txSignature3);

    // Task 4: Create a new account, transfer 5,000 lamports to it, and transfer 7,000 lamports to the specified account
    const newAccount3 = createNewAccount();
    console.log('New Account 3:', newAccount3.publicKey.toBase58());
    const transaction4 = new solanaWeb3.Transaction()
        .add(
            solanaWeb3.SystemProgram.transfer({
                fromPubkey: feePayer.publicKey,
                toPubkey: newAccount3.publicKey,
                lamports: 5000,
            })
        )
        .add(
            solanaWeb3.SystemProgram.transfer({
                fromPubkey: feePayer.publicKey,
                toPubkey: specifiedAccount,
                lamports: 7000,
            })
        );
    const txSignature4 = await solanaWeb3.sendAndConfirmTransaction(connection, transaction4, [feePayer]);
    console.log('Create new account, transfer 5,000 lamports to it, and transfer 7,000 lamports to specified account:', txSignature4);
};

// Execute the main function
main().catch(err => {
    console.error(err);
});
