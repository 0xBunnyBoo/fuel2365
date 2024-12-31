function YourPage() {
  // It is important to memoize the function call object.
  const incrementFunction = useMemo(() => {
    if (!contract || !wallet) return undefined;
    return contract.functions.increment_counter(amount);
  }, [contract, wallet, amount]);

  const { preparedTxReq, reprepareTxReq } =
    usePrepareContractCall(incrementFunction);

  const onIncrementPressed = async () => {
    if (preparedTxReq) {
      // Use the prepared transaction request if it is available. Much faster.
      await wallet.sendTransaction(preparedTxReq);
    } else {
      // Fallback to the regular call if the transaction request is not available.
      await contract.functions.increment_counter(1).call();
    }

    /*
     * Reprepare the transaction request since the user may want to increment again.
     * This is important, since the old `preparedTxReq` will not be valid anymore
     * because it contains UTXOs that have been used among other things.
     *
     * You *must* re-prepare any prepared transaction requests whenever
     * the user does any transaction.
     */
    await reprepareTxReq();
  };
}
