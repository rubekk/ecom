<script>
    import { onMount } from "svelte";
    import { db, auth } from "$lib/firebaseConfig"; 
    import { onAuthStateChanged } from "firebase/auth";
    import { collection, query, where, getDocs } from "firebase/firestore";
    import { authStore } from "$lib/store";

    let sAuthStore = { loggedIn: false, user: null };
    let orderHistory = [];
    let loading = true;

    authStore.subscribe(value => {
        sAuthStore = value;
    });

    const getOrderHistory = async () => {
        if (sAuthStore.loggedIn && sAuthStore.user) {
            try {
                const ordersRef = collection(db, "orders");
                const q = query(ordersRef, where("orderUserId", "==", sAuthStore.user.uid));
                const querySnapshot = await getDocs(q);

                orderHistory = querySnapshot.docs.map(doc => doc.data());

                console.log(orderHistory);
                loading = false;
            } catch (error) {
                console.error("Error fetching order history:", error);
                loading = false;
            }
        }
    };

    onMount(() => {
        onAuthStateChanged(auth, (currentUser) => {
            if (currentUser) {
                sAuthStore.loggedIn = true;
                sAuthStore.user = currentUser;
                getOrderHistory();
            } else {
                sAuthStore.loggedIn = false;
                sAuthStore.user = null;
            }
        });
    });
</script>

{#if !sAuthStore.loggedIn}
    <div class="loading-container">
        <p>Login to view your orders...</p>
    </div>
{:else if loading}
    <div class="loading-container">
        <p>Loading your order history...</p>
    </div>
{:else}
    {#if orderHistory.length === 0}
        <div class="no-orders">
            <p>You haven't placed any orders yet.</p>
        </div>
    {:else}
        <div class="order-history-container">
            {#each orderHistory as order}
            <div class="order-card">
                <div class="order-header">
                    <span class="order-date">{order.orderDate}</span>
                    <span class="order-time">{order.orderTime}</span>
                </div>
                <table class="order-items">
                    <thead>
                        <tr>
                            <th>Item</th>
                            <th>Qty</th>
                            <th>Price</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each order.orderProducts as product}
                        <tr>
                            <td>{product.productName}</td>
                            <td>{product.quantity}</td>
                            <td>Rs {product.price}</td>
                        </tr>
                        {/each}
                    </tbody>
                </table>
                <div class="order-total">
                    <strong>Total: Rs {
                        order.orderProducts.reduce((total, product) => total + (product.price * product.quantity), 0)
                    }</strong>
                </div>
            </div>
            {/each}
        </div>
    {/if}
{/if}

<style>
    .loading-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        font-size: 1.5rem;
        color: #666;
    }

    .no-orders {
        text-align: center;
        margin-top: 3rem;
        font-size: 1.2rem;
        color: #666;
    }

    .order-history-container {
        padding: 2rem;
        display: flex;
        flex-wrap: wrap;
        gap: 1.5rem;
        justify-content: center;
    }

    .order-card {
        width: 400px;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        background-color: #fff;
        border: 1px solid #eaeaea;
        display: flex;
        flex-direction: column;
        margin: 0; /* Remove margin to prevent stretching */
    }

    .order-header {
        display: flex;
        justify-content: space-between;
        margin-bottom: 1rem;
        font-size: 1rem;
        color: var(--blue);
    }

    .order-items {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 1rem;
    }

    .order-items th, .order-items td {
        padding: 0.5rem;
        border-bottom: 1px solid #eaeaea;
        text-align: left;
    }

    .order-items th {
        background-color: var(--blue);
        color: #fff;
    }

    .order-items td {
        color: #555;
    }

    .order-total {
        text-align: right;
        font-size: 1.2rem;
        color: var(--green);
    }
</style>
