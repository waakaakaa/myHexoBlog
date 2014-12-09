title: 'Android ListView最佳处理方式'
date: 2014-12-09 00:12:16

---
转自：[http://blog.csdn.net/boylinux/article/details/8860443](http://blog.csdn.net/boylinux/article/details/8860443)

Android ListView最佳处理方式，ListView拖动防重复数据显示，单击响应子控件。

<!--more-->

1. 为了防止拖动ListView时，在列表末尾重复数据显示。需要加入 HashMap<Integer,View> lmap = new HashMap<Integer,View>();其中Integer为列表位置，View为子项视图，加入数据前首先if
 (lmap.get(position)==null) ，满足条件时，加入lmap.put(position, convertView);如果条件不满足，convertView = lmap.get(position);
2. 监听每个子控件时，一定要加入final int currentPosition=position;这样可以牢牢抓住每次点击的响应位置；最好把控件集成到类中。


		package logic;
		
		import java.util.HashMap;
		import java.util.List;
		import java.util.Map;
		
		import logic.PlaceAdapter.ViewHolder;
		
		import org.guiji.BigPictureActivity;
		import org.guiji.ClassUserListActivity;
		import org.guiji.CommentMoodActivity;
		import org.guiji.R;
		import org.guiji.UserHomeActivity;
		
		import support.ClassUserListImageDownloadTask;
		import support.ImageDownloadTask;
		import support.SystemConstant;
		import android.content.Context;
		import android.content.Intent;
		import android.text.Html;
		import android.view.LayoutInflater;
		import android.view.View;
		import android.view.ViewGroup;
		import android.view.View.OnClickListener;
		import android.widget.BaseAdapter;
		import android.widget.ImageView;
		import android.widget.TextView;
		
		public class ClassUserListAdapter extends BaseAdapter {
		    HashMap<Integer,View> lmap = new HashMap<Integer,View>();
		    private LayoutInflater mInflater=null;
		    private List<Map<String, String>> mData=null;
		    private ClassUserListImageDownloadTask imgtask=null;
		    private Context context;
		    public List<Map<String, String>> getmData() {
		        return mData;
		    }
		
		    public void setmData(List<Map<String, String>> mData) {
		        this.mData = mData;
		    }
		    public ClassUserListAdapter(Context context){
		        this.mInflater = LayoutInflater.from(context);
		        this.context=context;
		    }
		    @Override
		    public int getCount() {
		        return mData.size();
		    }
		
		    @Override
		    public Object getItem(int position) {
		        return mData.get(position);
		    }
		
		    @Override
		    public long getItemId(int position) {
		        return position;
		    }
		
		    @Override
		    public View getView(int position, View convertView, ViewGroup parent) {
		        ClassUserListViewHolder holder = null;
		        if (lmap.get(position)==null) {
		            imgtask=new ClassUserListImageDownloadTask();
		            
		            convertView = mInflater.inflate(R.layout.classuserlist_item, null);
		            holder=new ClassUserListViewHolder(); 
		            	holder.classuserlist_item_userlogo=(ImageView)convertView.findViewById(R.id.classuserlist_item_userlogo);
		            holder.classuserlist_item_username=(TextView)convertView.findViewById(R.id.classuserlist_item_username);
		            holder.classuserlist_item_statuscontent=(TextView)convertView.findViewById(R.id.classuserlist_item_statuscontent);
		            holder.classuserlist_item_idfans1=(TextView)convertView.findViewById(R.id.classuserlist_item_idfans1);
		            holder.classuserlist_item_idfans2=(TextView)convertView.findViewById(R.id.classuserlist_item_idfans2);
		            holder.classuserlist_item_idmood=(TextView)convertView.findViewById(R.id.classuserlist_item_idmood);
		            holder.classuserlist_item_idhuoyuevalue=(TextView)convertView.findViewById(R.id.classuserlist_item_idhuoyuevalue);
		            holder.classuserlist_item_msgpic=(ImageView)convertView.findViewById(R.id.classuserlist_item_msgpic);
		            holder.classuserlist_item_msgcontent=(TextView)convertView.findViewById(R.id.classuserlist_item_msgcontent);
		            holder.classuserlist_item_idtimeplace=(TextView)convertView.findViewById(R.id.classuserlist_item_idtimeplace);
		            holder.classuserlist_item_classbutton=(ImageView)convertView.findViewById(R.id.classuserlist_item_classbutton);
		            
		            lmap.put(position, convertView);
		            convertView.setTag(holder);
		            holder.classuserlist_item_username.setText((String)mData.get(position).get("username"));
		            if(mData.get(position).get("idstatuscontent")!=null){
		                holder.classuserlist_item_statuscontent.setText((String)mData.get(position).get("idstatuscontent"));
		                holder.classuserlist_item_statuscontent.setVisibility(0);
		            }
		            if(mData.get(position).get("idfans1")!=null){
		                holder.classuserlist_item_idfans1.setText((String)mData.get(position).get("idfans1"));
		                holder.classuserlist_item_idfans1.setVisibility(0);
		            }
		            if(mData.get(position).get("idfans2")!=null){
		                holder.classuserlist_item_idfans2.setText((String)mData.get(position).get("idfans2"));
		                holder.classuserlist_item_idfans2.setVisibility(0);
		            }
		            
		            holder.classuserlist_item_idmood.setText((String)mData.get(position).get("idmood"));
		            if(mData.get(position).get("idhuoyuevalue").length()>=4)
		                holder.classuserlist_item_idhuoyuevalue.setText("活跃值"+"("+mData.get(position).get("idhuoyuevalue").substring(0, 4)+")");
		            else
		                holder.classuserlist_item_idhuoyuevalue.setText("活跃值"+"("+mData.get(position).get("idhuoyuevalue")+"0"+")");
		            
		            if(mData.get(position).get("idmsgcontent")!=null){
		                holder.classuserlist_item_msgcontent.setText((String)mData.get(position).get("idmsgcontent"));
		                holder.classuserlist_item_msgcontent.setVisibility(0);
		            }
		            if(mData.get(position).get("idtime")!=null){
		                holder.classuserlist_item_idtimeplace.setText((String)mData.get(position).get("idtime")+"  "+(String)mData.get(position).get("idplace"));
		                holder.classuserlist_item_idtimeplace.setVisibility(0);
		            }
		            
		            String temp=SystemConstant.baseURLNone+mData.get(position).get("userlogo")+","+
		            SystemConstant.baseURLNone+mData.get(position).get("idmsgpic");
		            imgtask.execute(temp,holder);
		            
		        }else {
		            convertView = lmap.get(position);
		            holder = (ClassUserListViewHolder)convertView.getTag();
		        }
		        
		        final int currentPosition=position;
		        holder.classuserlist_item_userlogo.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                MainService.guiji.setCurrentURL(SystemConstant.baseURL+mData.get(currentPosition).get("userLink"));
		                Intent it=new Intent(context,UserHomeActivity.class);
		                context.startActivity(it);
		            }
		        });
		        
		        holder.classuserlist_item_username.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                MainService.guiji.setCurrentURL(SystemConstant.baseURL+mData.get(currentPosition).get("userLink"));
		                Intent it=new Intent(context,UserHomeActivity.class);
		                context.startActivity(it);
		            }
		        });
		        holder.classuserlist_item_msgpic.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                MainService.guiji.setCurrentURL(SystemConstant.baseURLNone+mData.get(currentPosition).get("idmsgpic"));
		                Intent it=new Intent(context,BigPictureActivity.class);
		                context.startActivity(it);
		            }
		        });
		        holder.classuserlist_item_idfans1.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                MainService.guiji.deleteReply(SystemConstant.baseURL+mData.get(currentPosition).get("idfans1link"));
		                ((ClassUserListActivity) context).createTask();
		            }
		        });
		        holder.classuserlist_item_idfans2.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                MainService.guiji.deleteReply(SystemConstant.baseURL+mData.get(currentPosition).get("idfans2link"));
		                ((ClassUserListActivity) context).createTask();
		            }
		        });
		        
		        holder.classuserlist_item_classbutton.setOnClickListener(new OnClickListener() {
		            
		            @Override
		            public void onClick(View v) {
		                String temp=SystemConstant.baseURL+mData.get(currentPosition).get("idcommentlink");
		//                MainService.guiji.setCurrentURL(temp);
		                MainService.guiji.setPageType(4);
		                MainService.guiji.setBackURL(temp);
		                Intent it=new Intent(context,CommentMoodActivity.class);
		                context.startActivity(it);
		            }
		        });
		        
		        return convertView;
		    }
		    public class ClassUserListViewHolder{
		        public ImageView classuserlist_item_userlogo;
		        public TextView classuserlist_item_username;
		        public TextView classuserlist_item_statuscontent;
		        public TextView classuserlist_item_idfans1;
		        public TextView classuserlist_item_idfans2;
		        public TextView classuserlist_item_idmood;
		        public TextView classuserlist_item_idhuoyuevalue;
		        public ImageView classuserlist_item_msgpic;
		        public TextView classuserlist_item_msgcontent;
		        public TextView classuserlist_item_idtimeplace;
		        public ImageView classuserlist_item_classbutton;
		    }
		
		}

ListView在开发中是一个经常用的控件，有时候为了扩展更多功能也会用到ExpandableListView,然而数据的正确显示是开发者在实际开发中需要注意的，我在实现ListView的时候，最常遇到的是数据重复显示还有快速滑动的时候数据显示不完整的现象，然而如果在数据量大的时候还涉及到性能和convertView重用的问题。上面的这种处理方式，似乎已经比较好了，我已经验证过，之前滑动列表出现数据重复和显示不完整的现象全都没了。
