<template>
    <v-app>
        <v-container>
            <!-- Completed Orders Table -->
            <v-row>
                <v-col cols="12">
                    <!-- Skeleton Loader when data is being fetched -->
                    <v-skeleton-loader
                        v-if="loading"
                        type="table"
                        :columns="headers.length"
                        class="elevation-1"
                    />
                    <v-data-table v-else :headers="headers" :items="completedOrders" item-key="id" class="elevation-1">
                        <template v-slot:top>
                            <v-row>
                                <v-col cols="12">
                                    <v-card-title class="headline text-center">
                                        <v-icon class="mr-2">mdi-cube</v-icon>
                                        Completed Orders
                                    </v-card-title>
                                </v-col>
                            </v-row>
                        </template>

                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span>
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <!-- <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn> -->
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- <v-dialog v-model="dialog" max-width="500px">
                <v-card>
                    <v-card-title class="headline">Update Order Status</v-card-title>
                    <v-card-text>
                        <v-select v-model="selectedStatus" :items="statusOptions" label="Select New Status" dense
                            outlined></v-select>
                    </v-card-text>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn text color="red" @click="dialog = false">Cancel</v-btn>
                        <v-btn text color="green" @click="saveStatusUpdate">Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog> -->

            <v-dialog v-model="detailsDialog" max-width="600px">
                <v-card>
                    <v-card-title class="headline">Order Details</v-card-title>
                    <v-card-text>
                        <v-list dense>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>User:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.userId }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Products:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.productName }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Quantity:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.Quantity }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Subtotal:</v-list-item-title>
                                    <v-list-item-subtitle>₱{{ selectedOrderDetails?.subtotal }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Tax:</v-list-item-title>
                                    <v-list-item-subtitle>₱{{ selectedOrderDetails?.tax }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Total Amount:</v-list-item-title>
                                    <v-list-item-subtitle>₱{{ selectedOrderDetails?.total }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Payment Method:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.paymentMethod
                                        }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Estimated Delivery Date:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.estimatedDeliveryDate
                                        }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Delivery Address:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.deliveryAddress
                                        }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                            <v-list-item>
                                <v-list-item-content>
                                    <v-list-item-title>Status:</v-list-item-title>
                                    <v-list-item-subtitle>{{ selectedOrderDetails?.status }}</v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-list>
                    </v-card-text>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn text color="red" @click="detailsDialog = false">Close</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-container>
    </v-app>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { collection, query, where, getDocs, doc, getDoc, updateDoc } from 'firebase/firestore';

export default {
  data() {
    return {
      loading:true,
      completedOrders: [],
      headers: [
        { text: 'Order ID', value: 'id' },
        { text: 'Products Ordered', value: 'cartItems' },
        { text: 'User', value: 'userId' },
        { text: 'Quantity', value: 'totalQuantity' },
        { text: 'Total', value: 'totalAmount' },
        { text: 'Status', value: 'status' },
        { text: 'Payment Method', value: 'paymentMethod' },
        { text: 'Estimated Delivery Date', value: 'estimatedDeliveryDate' },
        { text: 'Created At', value: 'createdAt' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      dialog: false,
      detailsDialog: false,
      selectedStatus: '',
      selectedOrderDetails: null,
    //   statusOptions: ['Pending', 'Shipped', 'Delivered', 'Completed', 'Cancelled'],
      selectedOrderId: null,
    };
  },
  async created() {
    await this.fetchCompletedOrders();
  },
  methods: {
    async fetchCompletedOrders() {
      try {
        // Query for completed orders only
        const completedOrdersQuery = query(
          collection(firestore, 'Orders'),
          where('status', '==', 'Completed')
        );
        const querySnapshot = await getDocs(completedOrdersQuery);

        const orders = await Promise.all(
          querySnapshot.docs.map(async (docSnap) => {
            const orderData = docSnap.data();
            let userFullName = 'Unknown';

            // Fetch user details only if userId exists
            if (orderData.userId) {
              const userRef = doc(firestore, 'Users', orderData.userId);
              const userDoc = await getDoc(userRef);
              if (userDoc.exists()) {
                const userData = userDoc.data();
                userFullName = `${userData.firstName || ''} ${userData.lastName || ''}`;
              }
            }

            return {
              id: docSnap.id,
              ...orderData,
              totalAmount: orderData.totalAmount || this.calculateTotalAmount(orderData.cartItems),
              userFullName,
            };
          })
        );
        this.loading = false;
        this.completedOrders = orders;
      } catch (error) {
        this.loading = false;
        console.error('Error fetching completed orders:', error);
      }
    },
    // async saveStatusUpdate() {
    // try {
    //     const orderRef = doc(firestore, 'Orders', this.selectedOrderId);
    //     await updateDoc(orderRef, { status: this.selectedStatus });

    //     this.dialog = false;

    //     const updatedOrder = this.completedOrders.find(order => order.id === this.selectedOrderId);
    //     if (updatedOrder) {
    //     updatedOrder.status = this.selectedStatus;
    //     }
    //     this.fetchCompletedOrders();
    // } catch (error) {
    //     console.error('Error updating order status: ', error);
    // }
    // },
    formatDate(date) {
      const options = { year: 'numeric', month: 'short', day: 'numeric' };
      return new Date(date.seconds * 1000).toLocaleDateString('en-US', options);
    },
    calculateTotalQuantity(cartItems) {
      return cartItems.reduce((total, product) => total + (product.Quantity || 0), 0);
    },
    calculateTotalAmount(cartItems) {
      return cartItems.reduce(
        (total, product) => total + (product.price || 0) * (product.Quantity || 0),
        0
      );
    },
    // updateOrderStatus(order) {
    //   this.selectedOrderId = order.id;
    //   this.selectedStatus = order.status;
    //   this.dialog = true;
    // },
    viewOrderDetails(order) {
      this.selectedOrderDetails = {
        userId: order.userFullName,
        productName: order.cartItems.map((item) => item.productName).join(', '),
        Quantity: this.calculateTotalQuantity(order.cartItems),
        subtotal: order.cartItems.reduce((sum, item) => sum + (item.price * item.Quantity), 0),
        tax: order.cartItems.reduce((sum, item) => sum + (item.tax || 0), 0),
        total: order.totalAmount,
        paymentMethod: order.paymentMethod,
        estimatedDeliveryDate: order.estimatedDeliveryDate,
        status: order.status,
        deliveryAddress: order.deliveryAddress || 'Not Available',
      };
      this.detailsDialog = true;
    },
    getStatusColor(status) {
        // Assign a color based on the order status
        switch (status) {
        case 'Completed':
            return 'green';
        case 'Shipped':
            return 'blue';
        case 'Pending':
            return 'yellow';
        case 'Cancelled':
            return 'red';
        default:
            return 'grey';
        }
    },
  },
};
</script>



<style scoped>
.v-data-table {
    margin-top: 20px;
}
</style>