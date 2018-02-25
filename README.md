# broccoli
hello everybody 

please guys i need your help. im creating an app, and i want to display online users data (image and username) near a query Geofire, 
using a recyclerView. here is my code:


import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import com.bumptech.glide.Glide;
import com.firebase.geofire.GeoFire;
import com.firebase.geofire.GeoLocation;
import com.firebase.geofire.GeoQuery;
import com.firebase.geofire.GeoQueryEventListener;
import com.firebase.ui.database.FirebaseRecyclerAdapter;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.MutableData;
import com.google.firebase.database.Transaction;

import java.util.HashSet;


public class FetchedUsers extends AppCompatActivity {

    private FirebaseRecyclerAdapter<Post, PostViewHolder> mPostAdapter;
    private DatabaseReference mPostRef;

    private HashSet fetchedKeys = new HashSet();

    FirebaseAuth mAuth;

    GeoFire geofire;
    Double mLat;
    Double mLng;
    public GeoLocation latitudeLongitude;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fetched_users);

        initialiseScreen();

    }

    private void initialiseScreen() {

        RecyclerView mPostRV = findViewById(R.id.post_rv);
        mPostRV.setLayoutManager(new LinearLayoutManager(FetchedUsers.this));
        mPostRef = FirebaseDatabase.getInstance().getReference(Constants.POSTS);
        setupAdapter ();
        mPostRV.setAdapter(mPostAdapter);
    }


    private void setupAdapter() {

        mAuth = FirebaseAuth.getInstance();

        mPostAdapter = new FirebaseRecyclerAdapter<Post, PostViewHolder>(
                Post.class,
                R.layout.item_layout_post,
                PostViewHolder.class,
                mPostRef)  // come back here

        {
            @Override
            protected void populateViewHolder(PostViewHolder viewHolder, Post model, int position) {

                FirebaseUser user = mAuth.getCurrentUser();
                mLat = getIntent().getExtras().getDouble("LocationLat");
                mLng = getIntent().getExtras().getDouble("LocationLng");

                latitudeLongitude = new GeoLocation(mLat, mLng);

                GeoQuery geoQuery = geofire.queryAtLocation(latitudeLongitude, 10);
                geoQuery.addGeoQueryEventListener(new GeoQueryEventListener() {
                    @Override
                    public void onKeyEntered(String key, GeoLocation location) {

                        fetchedKeys.add(key);
                    }

                    @Override
                    public void onKeyExited(String key) {
                        fetchedKeys.remove(key);

                    }

                    @Override
                    public void onKeyMoved(String key, GeoLocation location) {
                        fetchedKeys.add(key);
                    }

                    @Override
                    public void onGeoQueryReady() {

                        if(user != null && fetchedKeys.contains(user.getUid()))
                        {
                            if(user.getPhotoUrl() != null){
                                Glide.with(FetchedUsers.this)
                                        .load(user.getPhotoUrl().toString())
                                        .into(viewHolder.postIV);
                            }
                        }

                        viewHolder.setNumViewers(model.getNumViewers());
                        viewHolder.postIV.setOnClickListener(v -> updateNumViewers(user.getPhotoUrl().toString()));

                        if (fetchedKeys.isEmpty()){
                            Toast.makeText(FetchedUsers.this, "no matches founded", Toast.LENGTH_SHORT).show();
                        }
                    }

                    @Override
                    public void onGeoQueryError(DatabaseError error) {

                    }
                });
            }
        };
    }

    private void updateNumViewers(String uid) {

        mPostRef.child(uid).child(Constants.Num_Viewers)
                .runTransaction(new Transaction.Handler() {
                    @Override
                    public Transaction.Result doTransaction(MutableData mutableData) {
                        long num = (long) mutableData.getValue();
                        num++;
                        mutableData.setValue(num);
                        return Transaction.success(mutableData);
                    }

                    @Override
                    public void onComplete(DatabaseError databaseError, boolean b, DataSnapshot dataSnapshot) {
                    }
                });

    }

    public static class PostViewHolder extends RecyclerView.ViewHolder{

         ImageView postIV;
         ImageView postViewersIV;
         TextView numViewersTV;

       public PostViewHolder(View itemView) {
           super(itemView);

           postIV  = itemView.findViewById(R.id.post_iv);
           postViewersIV = itemView.findViewById(R.id.viewers_tv);
           numViewersTV = itemView.findViewById(R.id.num_viewers_tv);
       }

       public void setNumViewers(Long num){
           numViewersTV.setText(String.valueOf(num));
       }

   }
}

if anyone can help me i'll be so grateful

